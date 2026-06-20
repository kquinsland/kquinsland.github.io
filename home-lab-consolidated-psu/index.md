# home lab: consolidating multiple PSUs

Canonical: https://karlquinsland.com/home-lab-consolidated-psu/
Published: 2020-06-27
Updated: 2020-06-27
Tags: fusion-360, has:download


---

A while back, I traded writing salt states and managing systemd `.service` files wrapping `podman` for the ['simplicity'](https://www.youtube.com/watch?v=9wvEwPLcLcA) of running kubernetes in my home lab. Jury is still out on weather or not the switch was worth it.

The cluster is a hodgepodge of second hand Intel NUCs and other scavenged compute hardware i've collected over the past few years. Some of it runs on 12v, some of it on 19v. For each node, there's a dedicated switch mode power supply. Each supply takes up an outlet and brings a bit of cable mgmt related clutter. The solution is consolidation.

So thats how this project came to be.

## What

A set of 3d printable parts to compactly combine two LRS-350 PSUs into a compact unit with optional voltage and current displays.

The details are on the [thingiverse](https://www.thingiverse.com/thing:4504948) and [prusaprinters](https://www.prusaprinters.org/prints/35514-dual-mean-well-lrs-350-psu) page.

## Files

For archive / posterity, all source files are included here. All files are licensed as [Creative Commons Attribution-NonCommercial-ShareAlike](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA).

- [images/built.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/built.webp)
- [images/cad-back.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/cad-back.webp)
- [images/cad-front.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/cad-front.webp)
- [images/core.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/core.webp)
- [images/meter-panel.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/meter-panel.webp)
- [Consolidated PSU CAD](https://karlquinsland.com/home-lab-consolidated-psu/models/2x-lrs-psu.f3z): Fusion360 Archive
- [Box](https://karlquinsland.com/home-lab-consolidated-psu/models/box.stl)
- [Brace](https://karlquinsland.com/home-lab-consolidated-psu/models/brace.stl)
- [Core Components](https://karlquinsland.com/home-lab-consolidated-psu/models/core.3mf)
- [Lid](https://karlquinsland.com/home-lab-consolidated-psu/models/lid.stl)
- [(Optional) Meter Panel](https://karlquinsland.com/home-lab-consolidated-psu/models/meter-panel.3mf)
- [(Optional) Meter Panel](https://karlquinsland.com/home-lab-consolidated-psu/models/meter-panel.stl)
- [images/meter-panel-slice.webp](https://karlquinsland.com/home-lab-consolidated-psu/images/meter-panel-slice.webp)


