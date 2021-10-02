# CAPP: Cloud Agnostic Proxy Protocol


It's another "i made a thing!" post. 🙃

-----

I'm playing around with porting a few different applications over to k8. Some of them - like [skyhole](https://github.com/kquinsland/skyhole/) -  rely on UDP packets which my hosted k8 provider of choice (read: cheapest! :moneybag:) [does not support](https://ideas.digitalocean.com/ideas/DO-I-310).

The solution is either:

1. Keep track of all of the node IP addresses that have an exposed `nodePort` and have the client connect directly to the cluster on the `nodePort`

2. Set up some sort of intermediary to re-write the packet from the `targetPort` **to** the `nodePort`

Option 1 seems fragile; if a cluster host / node is recycled, the client needs to learn the new correct IP addresses.

Option 2 seems far more flexible and is basically how most hosted k8 providers do ingress already. In fact, if digital ocean _had_ UDP support, this tool would never have been created. Bonus: clients that expect a specific port can still connect to the service no matter what `nodePort` is used.

So that's what [`CAPP`](https://github.com/kquinsland/cloud-agnostic-protocol-proxy) is: **C**loud **A**gnostic **P**rotocol **P**roxy.

It's a repo w/ a small python tool which can collect the IP addresses(s) belonging to each node in an arbitrary k8 cluster and - when combined with a small  user config file - generate configuration files for `traefik`.

A 'reference' packer job is included to generate a basic machine image to host traefik. A few lines of `cloud_init` is all that's needed to kick the tool off.

