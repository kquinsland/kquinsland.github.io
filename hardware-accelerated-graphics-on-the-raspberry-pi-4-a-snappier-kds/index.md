# Hardware accelerated graphics on the raspberry pi4 for a speedier KDS


Surfacing the right information at the right time in the right place is a difficult but essential task for any credible automation system.

I have been experimenting with the concept of a Home Assistant powered [KDS](https://www.qsrautomations.com/blog/kitchen-automation/what-is-a-kds/) for a few years now and have found that the refrigerator happens to be an especially good place to surface some information and device controls.

<br>

{{<figure name="kds-feature">}}

<br>

This **is not** a post about how to build the KDS pictured above; you can follow any of the numerous "how to use a raspberry pi as a kiosk" guides out there and you'd be 90% of the way there.
The last 10% is highly application specific; the hardware *you* design and 3d-print or otherwise fabricate will depend on your particular screen and selected mounting location.

This **is** a post about solving a performance issue with the dashboard hosted on the device.
Earlier versions of this KDS were powered by a raspberry pi 3 B (not even the `+` variant!) with 1 GB of ram.
The [Lovelace](https://www.home-assistant.io/lovelace/) dashboard that the KDS displays is quite javascript heavy and would frequently hang on the pi3b.

I use this dashboard to adjust grocery lists, control lights, set appliance timers and more. If the dashboard is sluggish or otherwise unresponsive, it's worse than useless!

As soon as I swapped in a raspberry pi 4 with 4 GB ram, things got .... marginally better 😑.

Sure, it booted much faster and the JS heavy graphs didn't cause the box to wedge anymore, but manipulating things on screen still was still a choppy experience.
Animations didn't show at all or had all but a few frames dropped :(.

I know that the [raspberry pi 4 is more than capable](https://medium.com/@ghalfacree/benchmarking-the-raspberry-pi-4-73e5afbcd54b) of running a single web page in a headless browser so _something_ is wrong.

After a bit of digging, it turns out that chromium does NOT use hardware acceleration by default on the raspberry pi.


Ok, that's an easy fix. Just enable the gpu and reboot:

```shell
pi@kds:~ $ cat /boot/config.txt 
<...>
# Enable DRM VC4 V3D driver for much more performant chrome
dtoverlay=vc4-kms-v3d
<...>
pi@kds:~ $ sudo reboot
```

... right?

**No.**

Turning on the gpu acceleration broke the various screen rotation / resolution directives that I had configured in `/boot/config.txt`:

```shell
pi@kds:~ $ cat /boot/config.txt 
<...>
[all]
# We need to rotate the display 90degrees as the 'default' orientation from the manufacturer assumes a horizontal orientation, not a vertical one
# See: https://www.waveshare.com/wiki/13.3inch_HDMI_LCD_(H)
display_rotate=1

# Force correct resolution
# Use 'DMT/Display Monitor Timings
# See: https://www.raspberrypi.com/documentation/computers/config_txt.html#video-options
hdmi_group=2
hdmi_mode=82
```

One of the Raspberry Pi Engineers [explains why](https://forums.raspberrypi.com/viewtopic.php?t=267954#p1626774):

> display_rotate only does anything if the firmware is in charge of the display, which isn't the case when using vc4-kms-v3d.

Even when the video out could be configured with `config.txt`, the firmware offers no such mechanism to rotate input events to match the orientation.
Fortunately, the LCD manufacturer [provides good documentation about how to configure the touch inputs with X11](https://www.waveshare.com/wiki/13.3inch_HDMI_LCD_(H)#Rotation.28Working_with_Raspberry_Pi.29)

Unfortunately, the manufacturer _does **not**_ provide corresponding documentation about how to configure the display with X11. Not that I can blame them! Configuring X11 has always been... tedious ... to use a 'polite' term for the experience!

## Xorg and SSH

I'll spare you the bulk of the rant and summarize with this:

```
Can't open display :0.0
```

For reasons that don't make a ton of sense to me, _all_ of the command line tools for probing display hardware and creating X11 configurations really don't like working over SSH 🤔. Yes, I of course tried the `export DISPLAY=:0.0` [trick](https://www.linuxquestions.org/questions/linux-general-1/xrandr-from-remote-through-ssh-869084/page2.html).


`xrandr` and friends are fine when running in a _local_ shell, but just don't play nice when a local user connects via SSH.
This makes things a lot harder than they needed to be as the location of the KDS does not lend its self to easily hooking up a keyboard and mouse.

Furthermore, the _intended_ purpose of this install is to display a web page. **Thats it.**
There is next to no desktop environment installed because one is not needed for a full screen headless chrome instance.

Installing a virtual console application was going to involve a lot of bloat and other unnecessary packages which is _overkill_ for a few CLI utilities that **should just work over any console weather that be local PTY or ssh!**

After a few hours of anguish and trial/error later, I had everything working as expected! But before we get there and while i'm still ranting, [why is there no `xorg -checkconf ...` command](https://unix.stackexchange.com/questions/435702/check-syntax-of-conf-file-in-etc-x11-xorg-conf-d)?!

Ok. Now it's all out of my system.

`</rant>`


## Solution


Every tap registers instantly and there's no jank or stutter in any animation. Likewise, graphs animate as quickly as they load... Perfect!

This is the X11 configuration that rotated the display and would 'see' the display attached to the GPU:

```shell
pi@kds:~ $ cat /etc/X11/xorg.conf.d/monitor.conf
# Device = GPU
# When using rPi 4 with GPU instead of software render, the `modesetting` driver is the one to use... apparently.
Section "Device"
        Identifier      "default"
        Driver          "modesetting"
EndSection

Section "Monitor"
        Identifier "default"
        # equivalent to `display_rotate = 1` in config.txt 
        Option "Rotate" "right"
EndSection

Section "Screen"
        Identifier "default"
        Monitor "default"
EndSection
```

In addition to the X11 config, some ram needs to be allocated as dedicated graphics memory:

```shell
pi@kds:~ $ cat /boot/config.txt | grep -B2 gpu_mem
[all]
# Allocate 256MB of ram for the GPU
gpu_mem=256
pi@kds:~ $ sudo reboot
```

Chrome did not automatically switch over to the GPU backed rendering pipe but it's easy enough to [configure chrome to use the GPU](https://lemariva.com/blog/2020/08/raspberry-pi-4-video-acceleration-decode-chromium) manually.


And with all that in place, a quick `sudo systemctl restart lightdm` and a brief screen flicker later, chromium launched in full screen mode with the correct orientation. After the dashboard finished loading, the scroll/tap/animation performance was as good as it would be on any competent computer!

I really don't know why I couldn't easily find a working X11 configuration example for use with the rPi 4 GPU, but I wasn't able to 🤷. 

Hopefully the above helps somebody else!

