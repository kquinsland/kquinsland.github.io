# home lab: consolidating multiple PSUs



A while back, I traded writing salt states and managing systemd `.service` files wrapping `podman` for the ['simplicity'](https://www.youtube.com/watch?v=9wvEwPLcLcA) of running kubernetes in my home lab. Jury is still out on weather or not the switch was worth it.

The cluster is a hodgepodge of second hand Intel NUCs and other scavenged compute hardware i've collected over the past few years. Some of it runs on 12v, some of it on 19v. For each node, there's a dedicated switch mode power supply. Each supply takes up an outlet and brings a bit of cable mgmt related clutter. The solution is consolidation.

So thats how this project came to be.

## What

A set of 3d printable parts to compactly combine two LRS-350 PSUs into a compact unit with optional voltage and current displays.

The details are on the [thingiverse](https://www.thingiverse.com/thing:4504948) and [prusaprinters](https://www.prusaprinters.org/prints/35514-dual-mean-well-lrs-350-psu) page. 

## Files

For archive / posterity, all source files are included here. All files are licensed as [Creative Commons Attribution-NonCommercial-ShareAlike](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA).

{{< bundle-files >}}

