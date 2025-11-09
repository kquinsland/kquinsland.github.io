# PoE at a distance caused terrible cat5 speeds


This is a quick post for made in the hopes that some poor soul in the future will find it and save themselves some time.

## Background

I've been experimenting with a few different ways to surface various home automation controls in the appropriate place and at a good time. One prototype host is deployed behind a small LCD under some cabinets in a high traffic area. This host is a raspberry pi 4 with a [PoE hat](https://www.raspberrypi.org/products/poe-hat/). It boots a very stripped down version of [Raspberry Pi OS](https://www.raspberrypi.org/downloads/raspberry-pi-os/) into a web browser running in Kiosk mode.

It's pretty simple and does many things well. There's a few small issues that need to be addressed in the CAD files as well as some additional electronic improvements that should be made before I can publish the design / files.

In any event, this host has been trouble-free until recently. Simple `apt update` transfers would take **DAYS** to complete, simple `scp` operations to other hosts on the LAN would trickle by at tens of kB/s. The web app that the host is primarily responsible for displaying would time out or fail to completely load resulting in some "a-for-effort" [graceful degradation](https://www.mavenecommerce.com/2017/10/31/progressive-enhancement-vs-graceful-degradation/).

## The Facts

As soon as I bring up the `wlan0` interface and bring down the `eth0` interface performance is IMMEDIATELY restored. The web app is responsive and fully loads instantly, `scp` is fast like it should be on a LAN and `apt update` completes without any errors as fast as my internet connection will let it!

**Conclusion**: The problem is *not* network related. Both the `eth0` and `wlan0` interface are on the same `vLan` so the bizarre behavior is not likely caused by a QoS policy or firewall rule.

The rPi and a few other hosts a powered via a a [120 watt](https://shop.poetexas.com/products/at-4-48v120w/) which is in the same closet as my core switch. Two terminations and about 200 feet of cat5 cable later, you get to the rPi host on the other side of the house.

Continuity tester was used on every segment of the link to confirm the link is good. The segments of the link from the switch to the injector and to the patch-panel all use high quality pre-made cables, as does the final segment from the wall to the rPi. Shocker, they tested fine, just like the in-wall segment.

Additionally, no other host on the PoE injector is misbehaving. No other host on the PoE injector is indicating that there's not enough power, and the power supply feeding the injector isn't even warm.

**Conclusion**: The cables are certainly fine and the PoE injector is *probably fine*.

Next, I disabled the PoE injector all together and use the USBC port on the rPi for power. It was just as performant as with WiFi! I did not expect this result, but it did certainly narrow down the possible problem areas.

**Conclusion**: The problem is only present when the host is powered via PoE.

I was very confident that the PoE injector was not the problem, but in order to be certain, I used a second cheap PoE injector that lives on my bench power supply. Turns out, the same slow speeds manifested with the second injector, too.

**Conclusion**: The problem is not in the network, nor is it in the cables, nor is it in the injectors.

This only leaves one candidate: the host. Unfortunately, there's nothing about the host that screams "i'm the problem!". The microSD card is new and very fast. There's nothing in the logs indicating a hardware failure. On-device performance (via non-networking benchmarks) is inline with what i'd expect.

Here's two additional facts from some brief testing I did a few weeks ago. I had more or less forgotten about them and only really remembered that:

- When I tried a fresh install of Ubuntu on a fresh microSD card... same slow speeds.

- The host performs *perfectly* when plugged into other networks.

I was *confident* then, that the rPi was not the issue and that something about the network was. But I just spent 3 hours systematically testing out every component of the network and concluding that the network is fine!

**Conclusion**: I'm back where I started; angry and confused.

The "ah ha!" moment didn't come until i decided to try my second PoE injector *at the client*. Much to my surprise, **it worked!**.

After a bit more testing, I determined that the PoE HAT on the rPi is 'fine' when the PoE source is CLOSE... which probably explains the 'works fine on other networks' fact above.

That leaves just the 'problem is OS independent' fact which - after thoroughly vetting every part of the network and PoE system - makes it pretty obvious that the problem must be between the rPi and the cat5 cable coming from the wall. There's *only* the PoE HAT between the rPi and the cat5 cable coming from the wall. I borrowed a PoE HAT from another host elsewhere on the network and the problematic rPi started behaving again. Network transfer speeds were appropriate and the web app / dashboard was performant again!

**Conclusion**: the PoE HAT powering a home automation dashboard has failed in such a way that makes the `eth0` interface all but useless. replacing the HAT solved the problem.

## TL;DR

Several months ago, the home automation dashboard displayed on a PoE powered rPi 4 started to act up. Further investigation showed absolute crap network performance from the host to any other point on the LAN or WAN. After testing each component in isolation, the PoE HAT was the only reaming suspect. After further investigation, the failure mode is *not present* when the PoE source is *close* to the client, only when the source is far from the client.

So if you have a PoE powered rPi project out there w/ abysmal `eth0` performance, try replacing the PoE hat...

Also, if anybody knows *why* this happened, I'd love to know.

