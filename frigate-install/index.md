# Frigate From Scratch guide

<!-- markdownlint-disable-file MD002 -->

{{< admonition info "FYI" >}}
Portions of this post was edited for clarity with the help of [ChatGPT](https://chat.openai.com/).
{{< /admonition >}}

-----

The [Frigate NVR](https://frigate.video/) project is a relatively new entrant to the home security camera DVR space.
Like most immature yet popular software, it has a killer feature - very good object detection that just works™ and robust [Home Assistant integration](https://docs.frigate.video/integrations/home-assistant/).

Unfortunately, the Frigate docs are a bit spartan particularly around installing; they more or less start with "now that you've installed it, let's go over configuring / using ...".

While this does seem like a rather import omission, it's somewhat intentional.
This is because Frigate is only distributed as a Docker container so installing really boils down to "get a computer that runs `docker` and then make sure `docker run ...` is executed when you want".

Most of the existing guides out there all use `docker-compose` with only some minor attention paid to [supervisory](https://en.wikipedia.org/wiki/Supervisory_program) configuration:

- [FRIGATE NVR Project with Seeed Odyssey](https://wiki.seeedstudio.com/ODYSSEY-X86J4105-Frigate/)
- [Frigate NVR: Linux Manual Install](https://www.digimoot.com/frigate-nvr-linux-manual-install/)
- [installing frigate from scratch guide #4041](https://github.com/blakeblackshear/frigate/discussions/4041)

This document isn't going to introduce anything new or innovative but should offer an alternative that closely tracks the way I did it.

-----

## Goal

This post is going to cover the steps taken to get a host that:

- runs the Frigate docker container via [`systemd`](https://en.wikipedia.org/wiki/Systemd)
- uses a [coral.ai edge TPU](https://coral.ai/products/m2-accelerator-bm) for accelerated object detection
- records footage on a network mounted file share

This post is not going to cover details that are likely specific to your deployment or are whole posts on their own:

- host or camera hardware selection.
- camera placement
- host setup tasks like installing OS
- configuring frigate to use your specific cameras
- integrating frigate with Home Assistant

## Prep

I'm going to assume that you've already got a [suitable host](https://docs.frigate.video/frigate/hardware/) for running Frigate and that you have already set it up as to your liking; `ssh` keys, `$hostname` set, timezone and ntp servers set up ... etc.

I used a modern [Intel N6005](https://www.intel.com/content/www/us/en/products/sku/212327/intel-pentium-silver-n6005-processor-4m-cache-up-to-3-30-ghz/specifications.html) system running Ubuntu 22.04 but the general process should be very similar for you and may even be identical if you use a debian based OS on similar hardware.

### Install Docker

Installing the bare docker runtime is pretty straight forward. As this is a debian based host, I followed the [`apt` repo method](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

Make sure the system is up to date before installing anything new - if something goes wrong and breaks your system you'll have an easier time figuring out what needs fixing.

```shell
karl@nvr:~$ sudo -i
[sudo] password for karl:  
root@nvr:~# apt update; apt dist-upgrade -y; apt autoremove -y; apt autoclean -y
<...>
root@nvr:~# cat /var/run/reboot-required.pkgs  
linux-image-5.19.0-35-generic
linux-base
root@nvr:~# reboot
```

Post update, install a subset of the docker packages:

```shell
karl@nvr:~$ sudo mkdir -m 0755 -p /etc/apt/keyrings
karl@nvr:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
karl@nvr:~$ echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
karl@nvr:~$ sudo apt update; sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin
<...>
After this operation, 401 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
<...>
```

And then check signs of life:

```shell
karl@nvr:~$ sudo docker run hello-world
<...>
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
<...>
```

With that, docker is good to go and we can move on to the other prerequisites.

### Create Frigate User

It's good practice to create dedicated users with limited permissions to run the service(s) under.
Create the user and verify that the user does not need elevated credentials to talk to docker:

```shell
karl@nvr:~$ sudo useradd --comment "service user for Frigate NVR" --groups docker --system --shell /usr/bin/bash frigate
karl@nvr:~$ sudo -i
root@nvr:~# su - frigate
su: warning: cannot change directory to /home/frigate: No such file or directory
frigate@nvr:/root$ groups
frigate docker
frigate@nvr:/root$ docker container list -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
dd7a5d1694c7   hello-world   "/hello"   4 minutes ago   Exited (0) 4 minutes ago             mystifying_yalow
```

{{< admonition warning "Quite note about docker security" >}}
Yes, I am aware that - because the `frigate` user can invoke `docker` commands directly - it isn't difficult for the `frigate` user to [escalate credentials to those of `root`](https://docs.docker.com/engine/security/#docker-daemon-attack-surface).

Running the docker daemon in [rootless mode](https://docs.docker.com/engine/security/rootless/) or an alternative 'non-root' container management tool is one way to eliminate this risk but is beyond the scope of this post.

As always, [defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)); this frigate host is appropriately firewalled off from the rest of the network.
{{< /admonition >}}

After `docker` is installed and the `frigate` user is added to the `docker` group, the next pre-requisite is storage for Frigate recordings.

### Create Mounts

{{< admonition tip >}}
You can skip this step if you do not wish to use remote storage for the Frigate configuration and recordings.
{{< /admonition >}}

{{< admonition warning "Storage quotas" >}}
Frigate does not have [sophisticated controls](https://docs.frigate.video/configuration/record) for configuring how long recordings are kept so you are encouraged to set up a storage quota for whatever disk/mount/share you use for network recordings.

If you are going the network share route, the software on the NAS likely has this functionality.
If you are going with local storage, the simplest way to enforce a quota is to use a dedicated partition.

{{< /admonition >}}

Using [`.mount` files](https://www.freedesktop.org/software/systemd/man/systemd.mount.html), it is trivial to have systemd mount the network share before starting Frigate.
I chose to use a [NFS](https://en.wikipedia.org/wiki/Network_File_System) share as both the NAS and the Frigate host are *NIX based and file system permissions tend to work a lot cleaner over NFS compared to Samba.

If you run into errors related to the database that frigate uses, you may consider [re-locating the database to a local mount](https://docs.frigate.video/configuration/advanced#database).

The technique outlined below will work for Samba shares as well but the `.mount` files will be configured slightly differently and you'll need to install slightly different software:

```shell
karl@nvr:~$ sudo apt install nfs-common
# Only needed if you use SMB shares. Without this package, you will likely get obscure errors related to hostname resolution
# See: https://askubuntu.com/questions/373340/ubuntu-server-13-10-cant-mount-hard-drive-that-is-on-my-router/374699#374699
karl@nvr:~$ sudo apt-get install cifs-utils
```

With the correct smb/nfs packages installed, tell systemd how to mount the network share locally automatically:

```shell
# Create path on host
karl@nvr:~$ sudo mkdir -p /mnt/frigate
karl@nvr:~$ sudo chown -R frigate /mnt/frigate
karl@nvr:~$ ls -lah /mnt/frigate/
total 8.0K
drwxr-xr-x 2 frigate root 4.0K Mar  2 08:37 .
drwxr-xr-x 3 root    root 4.0K Mar  2 08:37 ..
# Create the mount files
karl@nvr:~$ sudo $EDITOR /etc/systemd/system/mnt-frigate.mount
karl@nvr:~$ sudo $EDITOR /etc/systemd/system/mnt-frigate.automount
# And apply them
karl@nvr:~$ sudo systemctl daemon-reload
karl@nvr:~$ sudo systemctl enable mnt-frigate.moun
karl@nvr:~$ sudo systemctl enable mnt-frigate.automount
karl@nvr:~$ sudo systemctl start mnt-frigate.mount
karl@nvr:~$ sudo systemctl start mnt-frigate.automount
```

The `/etc/systemd/system/mnt-frigate.mount` file looks like this:

```systemd
[Unit]
Description=Mount Frigate NFS share share locally for frigate docker container
Requires=systemd-networkd.service
After=network-online.target
Wants=network-online.target

[Mount]
What=yourServerIPHere:/path/to/your/nfs/share/here
Where=/mnt/frigate
Type=nfs
Options=default
TimeoutSec=5

[Install]
WantedBy=multi-user.target
```

And the `/etc/systemd/system/mnt-frigate.automount` looks like this:

```systemd
[Unit]
Description=automount for frigate

[Automount]
Where=/mnt/frigate
TimeoutIdleSec=0

[Install]
WantedBy=multi-user.target
```

After `systemctl start/enable ...` the mounts should be up and running.
Use `systemctl status` to check that things worked properly and use `journalctl` to check logs if things went wrong.
Ideally you'll have something that looks like this:

```shell
root@nvr:~# systemctl status mnt-frigate.mount
● mnt-frigate.mount - Mount Frigate NFS share share locally for frigate docker container
     Loaded: loaded (/proc/self/mountinfo; enabled; preset: enabled)
     Active: active (mounted) since <...>
TriggeredBy: ● mnt-frigate.automount
      Where: /mnt/frigate
       <...>
```

At this point, all the _basic_ pre-requisites are satisfied: a `frigate` specific user can run `docker run ...` commands and `systemd` will automatically mount the network share locally.

### Coral.ai edge TPU

{{< admonition tip  >}}
Skip this step if you are not using PCI-E based edge TPU nodes.
{{< /admonition >}}

As mentioned at the top, one of the features that makes Frigate so attractive is how easy it is to use dedicated hardware for image/object classification.

I am using a [PCI-Express](https://en.wikipedia.org/wiki/PCI_Express) based TPU so there's a little bit more wor required to successfully pass a PCIe device into a docker container.
Fortunately this process is a lot simpler than it used to be and the google provided [instructions](https://coral.ai/docs/m2/get-started/#2a-on-linux) are pretty clear:

```shell
# Confirm there is no apex driver present already.
karl@nvr:~$ sudo lsmod | grep apex
# Add apt repos
karl@nvr:~$ echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
deb https://packages.cloud.google.com/apt coral-edgetpu-stable main
# Install the driver
karl@nvr:~$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
<...>
karl@nvr:~$ sudo apt-get update; sudo apt-get install gasket-dkms libedgetpu1-std
Get:1 https://packages.cloud.google.com/apt coral-edgetpu-stable InRelease [6,332 B]
<...>
The following additional packages will be installed:
 build-essential bzip2 cpp cpp-12 dctrl-tools dh-dkms dkms dpkg-dev fakeroot fontconfig-config fonts-dejavu-core g++ g++-12 gcc
<...>
Do you want to continue? [Y/n] Y
<...>
```

After install, [`udev`](https://en.wikipedia.org/wiki/Udev) rules are needed to make sure the proper driver is loaded:

```shell
# Confirm file does not exist
karl@nvr:~$ sudo cat /etc/udev/rules.d/65-apex.rules
cat: /etc/udev/rules.d/65-apex.rules: No such file or directory
karl@nvr:~$ sudo sh -c "echo 'SUBSYSTEM==\"apex\", MODE=\"0660\", GROUP=\"apex\"' >> /etc/udev/rules.d/65-apex.rules"
# Add the `frigate` user to the `apex` group so it can access the "virtual" PCIe devices
karl@nvr:~$ sudo groupadd -U frigate apex
karl@nvr:~$ sudo groups frigate
frigate : frigate docker apex
# Cleanest way to make sure driver and udev rules work is to reboot
karl@nvr:~$ sudo reboot
```

Checking that the `apex` driver is properly loaded is quick and painless:

```shell
# I have a "dual" TPU so two devices show up. If you only have a "single" TPU, only one will show up.
karl@nvr:~$ sudo lspci -nn | grep 089a
[sudo] password for karl:  
03:00.0 System peripheral [0880]: Global Unichip Corp. Coral Edge TPU [1ac1:089a]
04:00.0 System peripheral [0880]: Global Unichip Corp. Coral Edge TPU [1ac1:089a]
karl@nvr:~$ sudo ls /dev/apex_*
/dev/apex_0 /dev/apex_1
```

And with that, all the core/extended pre-requisites are done and the remaining work is actually pretty minimal.

## Install

After the pre-requisites are satisfied so all that's left is the [`.unit` file](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) which will wrap the `docker run` commands.

### `EnvironmentFile`

Frigate supports env-var substitution in it's configuration file like so:

```yaml
mqtt:
  host: mqtt.server.com
  user: {FRIGATE_SOME_KEY_HERE}
```

The configuration file - sans sensitive information - can now be safely checked in to source control.
Create a file just for holding our env-vars:

```shell
karl@nvr:~$ sudo mkdir -p /etc/frigate
[sudo] password for karl:
karl@nvr:~$ sudo touch /etc/frigate/secrets
karl@nvr:~$ sudo chown -R frigate:frigate /etc/frigate
karl@nvr:~$ sudo chmod -R 0750 /etc/frigate
karl@nvr:~$ sudo ls -lah /etc/frigate/
total 12K
drwxr-x---   2 frigate frigate 4.0K Mar  4 16:13 .
drwxr-xr-x 110 root    root    4.0K Mar  4 15:16 ..
-rwxr-x---   1 frigate frigate  356 Mar  4 16:13 secrets
```

The secrets file is [simple key=value format](https://docs.docker.com/engine/reference/commandline/run/#env):

```shell
karl@nvr:~$ sudo cat /etc/frigate/secrets
# Note: frigate < 0.12 does not support env-var for mqtt.user; set it here/now for use in the future.
FRIGATE_MQTT_USER=frigate
FRIGATE_MQTT_PASSWORD=ChangeMe

FRIGATE_CAM01_RTSP_USER=frigate
FRIGATE_CAM01_RTSP_PASS=changeme
```

{{< admonition warning >}}
The use of env-var substitution for the `username` field in the MQTT section of the config requires frigate 0.12 or higher.
At the time of writing (2023.03), the latest `stable` release is `0.11`.

Some additional details in [this](https://github.com/blakeblackshear/frigate/issues/5640) GH issue.
{{< /admonition >}}

### Systemd Unit for Frigate

The `/etc/systemd/system/frigate.service` file:

```systemd
[Unit]
Description=Frigate NVR
# Don't start until after docker is healthy and the nfs share is mounted
After=docker.service
Requires=docker.service
Requires=mnt-frigate.mount

[Service]
User=frigate
Group=frigate

TimeoutStartSec=0
Restart=always

# We want to start with a fresh container every time
ExecStartPre=-/usr/bin/docker exec %n stop
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull blakeblackshear/frigate:stable

# Expose the web UI on port 80 to keep things a bit cleaner
ExecStart=/usr/bin/docker run --rm --name %n \
  --env-file /etc/frigate/secrets \
  --mount type=tmpfs,target=/tmp/cache,tmpfs-size=1000000000 \
  --device /dev/dri/renderD128 \
  --device /dev/apex_0:/dev/apex_0 \
  --device /dev/apex_1:/dev/apex_1 \
  --shm-size=64m \
  -v /mnt/frigate/storage:/media/frigate \
  -v /mnt/frigate/config:/config:ro \
  -v /etc/localtime:/etc/localtime:ro \
  -p 80:5000 \
  -p 1935:1935 \
  blakeblackshear/frigate:stable
[Install]
WantedBy=default.target
```

Then enable/start the service

```shell
karl@nvr:~$ sudo systemctl enable frigate.service
Created symlink /etc/systemd/system/default.target.wants/frigate.service → /etc/systemd/system/frigate.service.
karl@nvr:~$ sudo systemctl start frigate.service
<...>
karl@nvr:~$ sudo systemctl status frigate
● frigate.service - Frigate NVR
     Loaded: loaded (/etc/systemd/system/frigate.service; enabled; preset: enabled)
     Active: active (running) since <...>
     CGroup: /system.slice/frigate.service
             └─27104 /usr/bin/docker run --rm --name frigate.service --mount type=tmpfs,target=/tmp/cache,tmpfs-size=1000000000 --device /dev/dri/renderD128 --device /dev/apex_0:/dev/apex_0 --device /dev/apex_1:/dev/apex_1 --shm-size=>
```

And that's all there is to it :).

Hopefully that helps!

