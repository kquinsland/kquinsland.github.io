# Quick look inside two Ali Express mmWave presence detection sensors

Canonical: https://karlquinsland.com/two-mmwave-sensors/
Published: 2022-11-01
Updated: 2022-11-01
Tags: teardown


---

Millimeter Wave technology has recently hit "mass consumer product adoption" price points.
A casual search for "human presence sensor" on Ali Express will turn up a seemingly endless number of sub $40 devices that can detect movement far more accurately than any old PIR sensor.

Each listing is fairly generic; there's no explicit manufacturer details but they all use the same marketing images:

![bl_marketing](https://karlquinsland.com/two-mmwave-sensors/images/bl_ali_exp.webp)

![wh_marketing](https://karlquinsland.com/two-mmwave-sensors/images/wh_ali_exp.webp)


As the photos below will show, neither device is super well marked with a model number so I'll just refer to each by either the color of the enclosure or by the radar sensor inside.

Before getting into the specific modules, a brief look at the packaging for both.

## Packaging

The packaging isn't anything special but I am including pictures here in the off chance that some of the Mandarin markings is useful for somebody.

![package01_top](https://karlquinsland.com/two-mmwave-sensors/images/package01_top.webp)

![package02_btm](https://karlquinsland.com/two-mmwave-sensors/images/package02_btm.webp)

![package03_side1](https://karlquinsland.com/two-mmwave-sensors/images/package03_side1.webp)

![package03_side2](https://karlquinsland.com/two-mmwave-sensors/images/package03_side2.webp)

![package03_side3](https://karlquinsland.com/two-mmwave-sensors/images/package03_side3.webp)

![package03_side4](https://karlquinsland.com/two-mmwave-sensors/images/package03_side4.webp)


> [!WARNING] Micro USB in 2022?!
> Both devices featured here have _micro_ USB leads.
> It's 2022 and we're still using MICRO USB for power?
> I know that USB-C connectors are slightly more expensive but it just seems lazy and dated to put a micro USB port on any new design in 2022!


> [!NOTE] uart?
> I did probe both of the 4 pin connectors on the PCB but didn't see any signs of life.
> If those headers are for a UART, the MCU isn't responding to any inputs nor is it putting anything out over it :(.


<!-- markdownlint-disable-file MD002 -->
## `ZY-M100-S`

AKA "the white one". Box content is bare-bones; the USB lead is comically short for almost any "mid height on the wall" or ceiling installs so i'm not really sure what the point was.

![wh_in-box](https://karlquinsland.com/two-mmwave-sensors/images/wh_in-box.webp)


I'm not sure why they went with 3 screws here instead of 4. If you're going to omit _a_ screw for cost-cutting reasons, why not just use clips or a single screw in the middle?

![No, it's not the camera or the lighting, the text really is that blurry in real life.](https://karlquinsland.com/two-mmwave-sensors/images/wh_btm.webp)

_No, it's not the camera or the lighting, the text really is that blurry in real life._


The interior is more or less as expected; two highly integrated modules with a small MCU gluing them together.

![wh_interior](https://karlquinsland.com/two-mmwave-sensors/images/wh_interior.webp)


There is some evidence that the Zigbee version of the white sensor supports ambient brightness.
Based on the two small holes in the enclosure and the PCB markings, I suspect that `d2` is being used as a light sensor:
![wh_pcb_top](https://karlquinsland.com/two-mmwave-sensors/images/wh_pcb_top.webp)


Nothing of interest on the bottom of the PCB:
![wh_pcb_btm](https://karlquinsland.com/two-mmwave-sensors/images/wh_pcb_btm.webp)


![Close up of the sensor MCU](https://karlquinsland.com/two-mmwave-sensors/images/wh_pcb_ic_mcu.webp)

_Close up of the sensor MCU_


### Unique Markings

- PCB: `ZY-M100` and `V4.20`
- TuYa radio: [`WBR3`](https://developer.tuya.com/en/docs/iot/wbr3-module-datasheet?id=K9dujs2k5nriy)
- TuYa mcu: [`GD32E230`](https://www.gigadevice.com/products/microcontrollers/gd32/arm-cortex-m23/value-line/gd32e230-series/)
- Radar sensor: `JYSJ_5807_A01` with an IC marked `SJ 501`. ~~No results on this one. Please do [get in touch](/contact/) if you do know anything about it. I'd love to re-use the sensor in something else!~~

> [!NOTE] We have an ID!
> Thanks to the prolific [blakadder.com for bringing it to my attention](https://blakadder.com/zy-m100/) and [@crlogic](https://community.home-assistant.io/u/crlogic) from the Home Assistant community forums [for identifying the module _and_ linking to a datasheet](https://community.home-assistant.io/t/mmwave-wars-one-sensor-module-to-rule-them-all/453260/73)!
> 
> The module is [`Leapmmw 5.8GHz Motion Detection: MD5G20`](http://docs.leapmmw.com/%E4%BC%A0%E6%84%9F%E5%99%A8%E4%BA%A7%E5%93%81/%E6%A8%A1%E5%9D%97/module.html)


## The `MicRadar RD24D`

AKA "the black one".

Like the M100, there's not a ton in the box.

The rear enclosure has a simple key-hole slot for mounting to a wall or ceiling and the USB power lead is considerably longer than the M100. Somebody did consider that a celling-mounted device might need a long power lead...

![Yes, that is a neo-pixel LEDin the center of the opaque plastic panel.](https://karlquinsland.com/two-mmwave-sensors/images/bl_01_in-box.webp)

_Yes, that is a neo-pixel LEDin the center of the opaque plastic panel._

![bl_02_in-box](https://karlquinsland.com/two-mmwave-sensors/images/bl_02_in-box.webp)


Same story here: WiFi module and radar sensor both talk to a MCU over UART.

![bl_pcb-top](https://karlquinsland.com/two-mmwave-sensors/images/bl_pcb-top.webp)


Nothing remarkable on the bottom of the PCB:
![They only soldered in one half of the pins connecting the radar sensor to the main PCB. I wonder how much time/money that saved...](https://karlquinsland.com/two-mmwave-sensors/images/bl_pcb_btm.webp)

_They only soldered in one half of the pins connecting the radar sensor to the main PCB. I wonder how much time/money that saved..._


Here's a close up of the business end of the sensor. This one is considerably more sophisticated and complex relative to the white one.

![bl_sensor](https://karlquinsland.com/two-mmwave-sensors/images/bl_sensor.webp)


And a close up of the coordinating MCU

![bl_ic01-mcu](https://karlquinsland.com/two-mmwave-sensors/images/bl_ic01-mcu.webp)


<!-- markdownlint-disable-file MD024 -->
### Unique Markings

- PCB: `V1.01`
- TuYa radio: [`CB3S`](https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076)
- TuYa mcu: [STC 8G1K17](https://www.stcmicro.com/stc/stc8g1k08.html)
- Radar sensor: `MicRadar R24D`. I can't find a datasheet for the specific module, but it does seem to be part of [this product family](https://www.iflabel.com/product/28.html).
  - IC1: `MicRadar / T15BT / DAT2230`
  - IC2: This above picture isn't the best but the chip is marked with `S3KM11L / N46Y80D1 / 2123H`. The [_only_ google result](https://www.ic37.com/news/2022-3_293564/) does not really indicate _who_ makes the chip.

## Conclusion

Neither are going to be quick/easy to convert or otherwise get working with the likes of ESPHome or Tasmota:

- Neither wifi module is an espressif module. At least some re-flow work would be required to replace the modules with an Espressif module.
- There's a MCU sitting between the WiFi module and the actual sensor. Reverse engineering how the TuYa radio talks to the MCU and how the MCU talks to the radar module just isn't worth the time!

The sensors cost about as much as the bare radar modules themselves so - if you're willing to spend the time to desolder the radar modules - you get an enclosure for "free".

