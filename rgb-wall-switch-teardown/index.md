# Generic RGB Wall Switch teardown

Where 'generic' means 'SSMS118 clone'

Canonical: https://karlquinsland.com/rgb-wall-switch-teardown/
Published: 2022-01-01
Updated: 2022-01-01
Tags: tuya


---

Yep! Another teardown post! This one was also a cheap "because i'm curious" post.

----

# What

It's a 'decora' style wall switch with WiFI and an RGB color changing paddle.

![Picture from the product listing; device is rendered](https://karlquinsland.com/rgb-wall-switch-teardown/images/ali_listing.webp)


The product listing was very non specific:

> EU US WiFi Smart Wall Switch Push Button Timer Relay Switch Voice Remote Control RGB LED Night Light Lamp TUYA Alexa Google Home

I saw `TuYa` in the item description and hoped that it would be based on an ESP8266 module or at least use a pin-compatible module. After all, there are a few such devices on the [Wall Switches and Dimmers section](https://templates.blakadder.com/switch.html) of the tasmota templates repository.

## Teardown

Almost immediately after opening it became clear: the item is a variant of the [`SSMS118-01AI`](https://templates.blakadder.com/moko_SSMS118-01AI.html) switch.

Other than that, the internals are pretty predictable / standard.

From the side you can see plastic clips holding the paddle to the Aluminum substrate.

![Picture of the switch lying on its side. A seam is clearly visible showing how the two halves of the switch come together](https://karlquinsland.com/rgb-wall-switch-teardown/images/td01_side.webp)


No screws on the back, so we probably open it from the front.

![Picture of the switch lying on its front so the rear is visible. Product model number and other details are visible. No screws or other evidence that entry is gained from the rear.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td02_rear.webp)

_Not sure about how code compliant those wire terminal/lugs are._


If the paddle was removed, the philips screws would be unobstructed.

![Picture of the switch lying on it's rear so the face is visible. There are 4 slightly obstructed philips screws visible.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td03_front.webp)


![Picture of the switch lying on its side with the button surface partially removed from the main switch body.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td04_top.webp)


RGB leds on a thin PCB with a reflective sticker attached. The sticker probably makes the LEDs appear brighter.

![Picture of the switch with no paddle lying next to the paddle. A reflective PCB dotted with RGB leds is visible.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td05_no-face.webp)


Philips screws removed and the switch immediately splits in half. I wish everything was this easy to get into!

![Picture of ths switch body halves being separated.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td06_cracked.webp)


Almost textbook design at this point; keep the mains voltage on it's own PCB and only send back low voltage / signal to the controller PCB.

The mains PCB attaches to the controller PCB via 3 sets of three male/female pin headers along the top and bottom of the left edge and one side of the right edge.

![Picture of ths switch body halves with the internal PCBs facing out.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td07_split-faces.webp)


Sparsely populated and a generic no-name relay. But at least there's a fuse!

![Picture of the mains PCB and relay.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td08_mains-pcb.webp)


Mains PCB is marked `HYS-00-004-MAIN_V1.1` with a datecode of `2019-5-7`.

Equally sparse controller PCB. Aside from the _massive_ mechanical switch and the three transistors for the RGB leds, everything lives on the ESP module.
At first glance, the module looks like a standard ESP-12... but it isn't! There are 6 pins on each of the three edges, not the 8 that would be present on an ESP-12 module.

![Picture of the low voltage controller PCB and the LED PCB.](https://karlquinsland.com/rgb-wall-switch-teardown/images/td09_lv-pcb.webp)


Controller PCB is marked `HYS-00-004-WIFI_V1.1` with a datecode of `2019-4-11`.

After a bit of reverse image searching, the ESP based module appears to be a WT8266-S1s. The datasheet can be found [here](https://www.seeedstudio.com/document/word/WT8266-S1%20DataSheet%20V1.0.pdf).

To save you a click, here's the useful bit:

![Screenshot of the ESP module pinout](https://karlquinsland.com/rgb-wall-switch-teardown/images/esp-module-pinout.webp)


I checked a few of the pins listed on the  [`SSMS118-01AI template`](https://templates.blakadder.com/moko_SSMS118-01AI.html) switch and they appeared to match the PCB shown above.

Thats all for this one!

