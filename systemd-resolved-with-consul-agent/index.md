# Systemd Resolved With Consul Agent

get *.consul domains resolved from within docker containers without resorting to dnsmasq

Canonical: https://karlquinsland.com/systemd-resolved-with-consul-agent/
Published: 2020-07-21
Updated: 2020-07-21
Tags: consul, systemd, systemd-resolved


---


I pieced this technique together a while back and created a gist for it. I'm creating _this_ post as a pointer to that gist so I have something that's a bit easier to reference and refer others to.

And i want to test out the [hugo shortcode for embedding a gist](https://gohugo.io/content-management/shortcodes/#gist) :smirk:.

The really short version:

- Create a dedicated interface that can only be accessed from the local system
- Bind the consul-agent's DNS service to this local only interface
- Tell [`systemd-resolved`](https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html) that all hostnames with the `.consul` TLD can be resolved via a DNS server on this local interface

No need to disable `resolved` and replace it with dnsmasq :smile:

[GitHub Gist: kquinsland/5cdc63614a581d9b392f435740b58729](https://gist.github.com/kquinsland/5cdc63614a581d9b392f435740b58729)

