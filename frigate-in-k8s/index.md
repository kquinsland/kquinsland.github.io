# Frigate in Kubernetes

<!-- markdownlint-disable-file MD002 -->
# Frigate: Kubernetes edition

This is a follow up to my ['Frigate From Scratch' post]({{<relref "/posts/2023/03/frigate-install/">}}).
Most of that post is still relevant today but I have since moved almost all of my workloads into Kubernetes.

This post is intended to document some of the issues I hit while trying to get Frigate running in my k8s cluster.
This is not every issue I hit, just the ones that are not directly related to my specific environment/setup/workflows.

I have essentially taken the relevant parts of my notes and lightly edited them into a more organized and structured format.

{{<figure name="cover-frigate_logo">}}

## Overview

There is a [helm chart](https://github.com/blakeblackshear/blakeshome-charts) but Frigate - at least _outside_ of the container - is not a particularly complex deployment so there's no upside to using helm and just a **ton** of downside:

- [Helm 4\.0 | Hacker News](https://news.ycombinator.com/item?id=45902604)
- [\`helm\` is an absolute garbage\. I'll reserve judgment about \`kustomize\` until I h\.\.\.](https://news.ycombinator.com/item?id=24499272)
- [Helm local code execution via a malicious chart](https://news.ycombinator.com/item?id=44506696)
- [We use Helm, but we really only use it for two things: Templating and atomic dep\.\.\.](https://news.ycombinator.com/item?id=19580025)
- [5 shortcomings of Helm | Glasskube](https://glasskube.eu/en/r/knowledge/5-helm-shortcomings/)
- [Helm is problematic in my view\. To new Kubernetes users, Helm appears to be the \.\.\.](https://news.ycombinator.com/item?id=21526894)
- [Helm may be the most flagrant example of a terrible package that remains inexpli\.\.\.](https://news.ycombinator.com/item?id=11100312)
- [Why Helm’s design is flawed](https://blog.devops.dev/why-helms-design-is-flawed-a66c07c2e9a1?gi=6078bb173b62)

`</rant>`

You can still use the `helm` chart as a reference for the various ports and volume mounts that need to be configured:

```shell
❯ helm repo add blakeshome https://blakeblackshear.github.io/blakeshome-charts/
"blakeshome" has been added to your repositories
# use --set to enable things like startup probes and coral support to get a better reference manifest
❯ helm template blakeshome blakeshome/frigate --set probes.startup.enabled=true --set coral.enabled=true --set coral.hostPath=/dev/apex_0 > reference.yaml
```

## Pain Points

Below are two pain points I hit while getting Frigate running in k8s that are also likely to be relevant to others.
Hopefully the notes here will save you some time!

### Coral pass through

Frigate now supports _several_ different [hardware acceleration](https://docs.frigate.video/frigate/hardware/#detectors) options for object detection.
I am still using the PCIe Coral EdgeTPU as the performance is excellent although that will likely change in the future as other options become available.

To prepare for a future where I might have multiple viable hardware acceleration options available in my k8s cluster, I have elected to use [NFD (Node Feature Discovery)](#nfd) to label nodes that have Coral TPUs installed.
This is not particularly difficult to do _but it is also **unnecessary** if you only have a single node that will run Frigate_; regular taint/toleration or nodeSelector approaches will work just fine.

Regardless, the Coral drivers need to be installed on the node(s) that will host the Coral device(s).

The good news is that the process of manually patching and compiling the drivers has been made less tedious by the community!
So long as you are on a modern-ish Debian based system, the [jnicolson/gasket-builder repo](https://github.com/jnicolson/gasket-builder) has automated the process of installing the [`DKMS` drivers](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) for the Coral TPU.

You don't have to:

- track down a few unmerged patches and apply them
- figure out how to build the drivers against your current kernel
- manually load the modules
- create and load udev rules
- ... etc

You will still need kernel headers and basic build tools installed on the host but beyond that it's really just a matter of `wget ...; dpkg -i ...` and a `reboot now` for good measure.

Once the drivers are installed, you should see the devices show up under `/dev/apex_*`:

```shell
root@panopticon01:/mnt/frigate# ls -lah /dev/apex_*
crw-rw---- 1 root apex 120, 0 Dec 30 10:01 /dev/apex_0
crw-rw---- 1 root apex 120, 1 Dec 30 10:01 /dev/apex_1
```

Assuming the devices are detected, you can then pass them through to the container as regular `hostPath` volumes.
Unfortunately, there is no k8s equivalent of the [`--device` flag](https://docs.docker.com/reference/cli/docker/container/run/#device) and my attempts to use a tailored `securityContext` to grant the necessary permissions to access the devices failed:

```yaml
securityContext:
  privileged: false
  capabilities:
    add:
      # DMA access
      - IPC_LOCK
      # Required for PCIe memory access
      - SYS_RAWIO
      # Permit more things if needed
      - CAP_SYS_ADMIN
  # Allow anything that wants to escalate privileges to do so...
  allowPrivilegeEscalation: true
```

Kubernetes seems to prefer that hardware vendors create/ship a [Device Plugin](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/) to safely/cleanly facilitate accessing devices from inside pods but there is only a [very experimental one](https://gitlab.com/mike-ensor/k8s-edge-tpu-device-plugin) for the Coral TPU at this time.

All of that is to say: _you **will** need to run the container in `privileged` mode for the device access to work correctly_.

```yaml
apiVersion: apps/v1
kind: Deployment
# <...>
spec:
    template:
        spec:
            containers:
                  image: ghcr.io/blakeblackshear/frigate:0.16.3
                  name: frigate
                  securityContext:
                      # Needed for the Frigate UI to show CPU/TPU metrics (usage/temp...etc)
                      capabilities:
                          add:
                              - CAP_PERFMON
                      # See note above
                      privileged: true
                  volumeMounts:
                      # Covered in the IPv6 section below
                      - mountPath: /usr/local/nginx/templates/listen.gotmpl
                        name: nginx-templates
                        readOnly: true
                        subPath: listen.gotmpl
                      - mountPath: /dev/apex_0
                        name: coral-apex-0
                      - mountPath: /dev/apex_1
                        name: coral-apex-1
                    # <...>
            # Covered in the NFD section below
            nodeSelector:
                feature.node.kubernetes.io/has-pcie-tpu: "true"
            volumes:
                - configMap:
                      name: frigate-nginx-templates
                  name: nginx-templates
                - hostPath:
                      path: /dev/apex_0
                  name: coral-apex-0
                - hostPath:
                      path: /dev/apex_1
                  name: coral-apex-1
                # <...>
```

#### NFD

[Node Feature Discovery](https://github.com/kubernetes-sigs/node-feature-discovery) is a straight-forward project that puts a daemonset on each node in your cluster and scans the hardware for features.
Depending on what hardware it finds, it will label the node with appropriate labels that can then be used for scheduling.

{{< admonition warning "Over Engineered" >}}
As noted [above](#coral-pass-through), NFD is not strictly necessary if you only have a single node that will run Frigate, just use a regular `nodeSelector` or taint/toleration approach instead.
{{< /admonition >}}

I have some other workloads that can benefit from NFD so I went ahead and set it up cluster-wide.
Here is the relevant snippet from my NFD configuration that detects Coral TPUs and applies a label accordingly:

```yaml
core:
    # The worker is cheap but it defaults to running every 60s which is excessive for a cluster that
    #    is all bare metal and not likely to have hardware changes often.
    sleepInterval: 300s

    # all -> make sure `pci` features are included
    # Disable CPU features to reduce excessive labels
    featureSources:
        - all
        - "-cpu"

    labelSources:
        - all
        - "-cpu"

sources:
    pci:
        # You only need `0880` for Coral but I have some other hardware that I use
        deviceClassWhitelist:
            # Network
            - "02"
            # Display
            - "03"
            # Base class for System peripherals
            - "08"
            # Specific class for Coral
            - "0880"
            # Accelerator
            - "12"

    custom:
        - name: "has-coral-tpu"
          labels:
              "feature.node.kubernetes.io/has-pcie-tpu": "true"
          matchFeatures:
              - feature: pci.device
                matchExpressions:
                    vendor: { op: In, value: ["1ac1"] }
                    device: { op: In, value: ["089a"] }
```

Once NFD is deployed and running, each `kind: Node` in your cluster should have a label like so if it has a Coral TPU installed:

```shell
❯ kubectl get nodes -l feature.node.kubernetes.io/has-pcie-tpu -o name
node/panopticon01
```

Combine that with the `nodeSelector` as shown in the snippet \above and Frigate will only be scheduled on nodes that have Coral TPUs installed.

### IPv6

It's 2026 and IPv6 should just work by default everywhere but alas, Frigate is still a bit behind the times in this regard.

The [docs](https://docs.frigate.video/configuration/advanced/#enabling-ipv6) are very sparse but they do hint at having to make some modification to a file inside the container.

Above, I am providing my own copy of `/usr/local/nginx/templates/listen.gotmpl` via a `ConfigMap` volume mount to override the default one that ships with the container:

```yaml
kind: ConfigMap
# <...>
data:
    listen.gotmpl: |
        listen 5000;

        {{ if not .enabled }}
        # intended for external traffic, protected by auth
        listen [::]:8971 ipv6only=off;
        {{ else }}
        # intended for external traffic, protected by auth
        listen [::]:8971 ipv6only=off ssl;

        # intended for internal traffic, not protected by auth
        listen [::]:5000 ipv6only=off;

        {{ end }}
```

The above is more or less taken from [this GH issue](https://github.com/blakeblackshear/frigate/issues/5275#issuecomment-2395553577) but modified slightly:

- The GH issue uses `[::1]` for some reason which only binds to the loopback interface and does not work for me. I changed it to just `[::]` to bind to all interfaces which is what I want for my use case.
- The ACME/TLS stuff is not relevant to me so I removed that; I just handle TLS termination elsewhere in my stack.
- There's still a ton of stuff inside the container that is IPv4 only but that's a problem for another day / does not violate my self-imposed "IPv6 first, ideally exclusively" requirement.

## Conclusion

Frigate puts all the complexity into the container so it's a pretty straight-forward deployment in k8s.
Things get a bit trickier when you start adding hardware acceleration into the mix but it's still manageable with a bit of effort.
For reasons that are beyond me, Frigate still does not have first-class IPv6 support so that requires a bit of manual tinkering as well.

