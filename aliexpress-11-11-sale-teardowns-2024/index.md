# AliExpress 11.11 Sale Teardowns (2024 edition)

It's a whole Grab bag of worth of #TwoMinuteTeardowns.

Canonical: https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/
Published: 2024-12-01
Updated: 2024-12-01
Tags: two-minute-teardown


---

<!-- markdownlint-disable-file MD024 -->
# 11-11 day teardowns (2024 edition)

I guess this is becoming a [tradition](http://localhost:1313/categories/11.11-sale/) now?

During China's Single's Day sale on AliExpress, I bought various gadgets - both for projects and out of curiosity.
Rather than write separate posts, here's a collection of mini teardowns showing what's inside each item.

Previous years:

- [2023](/aliexpress-11-11-sale-teardowns/)

## Tuya Zigbee 3.5 Inch Smart Wall Switch

![01_tuya_switch_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807444250563.html?gatewayAdapt=glo2usa)

I'm always looking for a better scene controller/switch/Home Assistant interface.
I was curious about which chip they used for the full graphics display and was hoping for an easily hackable Android device.
Unfortunately, no such luck with this.

The main application processor is [`F1C100s`](https://linux-sunxi.org/F1C100s) which is a low-cost, low-power, ARM9-based SoC.
It might be able to run a custom Linux build that could draw a custom UI, but it's not going to be as easy as rooting an Android device.

There’s not much in the box, just the switch and a manual.
![01_tuya_switch_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td01.webp)


It's a pretty standard "power supply and relays in the back, touch screen in the front" design.
This time, though, there's an _exposed_ programming header!?

![Note the 'fire wire' label for mains input... :D](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td02.webp)

_Note the 'fire wire' label for mains input... :D_


A few screws later, the back panel comes off to reveal some power regulation/switching components and the Zigbee module.

![01_tuya_switch_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td03.webp)


They achieved the "mushy, then clicky" feel of the buttons by using a rubber insert around the tactile switches.

![01_tuya_switch_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td04.webp)


The power supply doesn't look too bad.
Clearance looks OK, there is a fuse, and the soldering is decent.
I don't know how well isolated that transformer is, but it's probably fine.

![01_tuya_switch_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td05.webp)

![J2 looks like it might be DC input or possibly RS485 for a future model?](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/01_tuya_switch/td06.webp)

_J2 looks like it might be DC input or possibly RS485 for a future model?_


## Ueetop 175W GaN USB C Charger (BK-112D)

![02_usb_charger_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807203418058.html?gatewayAdapt=glo2usa)

As the USB-C PD standard matures and GaN technology makes high-power chargers cheaper, it's fun to see the internals get more complex but also more integrated.
This was less than $15 shipped which is _insane_ for a 100W+ charger unless they cut corners somewhere...

![Hey, it came with a mains cord and it's even correct for my country!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td01.webp)

_Hey, it came with a mains cord and it's even correct for my country!_


There are a LOTS of chips holding things together; it's almost impossible to open this without breaking at least one!
Consider this a warning: you will be doing a destructive teardown if you open this!

![Look at those PCB traces going to the output. Easily 5mm wide!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td02.webp)

_Look at those PCB traces going to the output. Easily 5mm wide!_


As this is a [#TwoMinuteTeardown](/tags/two-minute-teardown/), I'm not going to remove all the silastic glue on the power supply module nor will I check the ripple in the output.

![02_usb_charger_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td03.webp)


Other than that, it's about what you'd expect from a modern USB-C charger that isn't hyper-compact; beefy looking AC-to-DC converter, a few big caps and then the USB-C PD controller.

Between each group of ports is a series of passives, some beefy looking power MOSFETs and a protocol negotiation IC.

![02_usb_charger_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td04.webp)

![02_usb_charger_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td05.webp)

![02_usb_charger_td06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td06.webp)

![02_usb_charger_td07](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/02_usb_charger/td07.webp)


### Notable ICs

[SW3516](https://ismartware.com/article-1-2.html)

[Shenzhen Fuman Elec XPD977D30A](https://www.lcsc.com/product-detail/USB-Converters_Shenzhen-Fuman-Elec-XPD977D30A_C2892062.html)

[CHIPSEA CSU32P10](https://www.lcsc.com/product-detail/Microcontroller-Units-MCUs-MPUs-SOCs_CHIPSEA-CSU32P10-SOP8_C914988.html)

## Generic Ozone generator (XD001)

![03_ozone_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256805759188983.html?gatewayAdapt=glo2usa)

There’s not much in the box other than the generator and a manual.

![03_ozone_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td01.webp)


It’s easy to take apart with no hidden screws!

![03_ozone_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td02.webp)


Inside, you’ll find about what you’d expect: a potted HV module, fan, controller, and a battery.

![03_ozone_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td03.webp)


Instead of the cheaper "carbon needle" ozone generators, this one uses a ceramic plate.

![Such a pretty glow!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td04.webp)

_Such a pretty glow!_


![Micro USB is INEXCUSABLE in 2024!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td05.webp)

_Micro USB is INEXCUSABLE in 2024!_


Annoyingly, they used a micro USB port for charging.
This is _inexcusable_ in 2024.
The silk screen on the PCB indicates that the design is from late 2020, which would be slightly more acceptable if USB-C wasn’t already the standard by then.
Furthermore, this device doesn’t need any complicated USB-C PD features, so it’s not like they’d have to worry about the extra cost or complexity of a USB-C controller.
Really, all they needed was _two_ (very important, [**NOT** ONE](https://hackaday.com/2019/07/16/exploring-the-raspberry-pi-4-usb-c-issue-in-depth/)) 5.1k resistors and a USB-C port and they could have been future proofed!

![03_ozone_td06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td06.webp)


They scrubbed the markings off the SOIC16 CPU. The rest of the components are about what you’d expect for managing lithium batteries, switching the fan, and driving the boost converter for the ozone generator.

It appears another version of this device also features a buzzer.
I'm not sure what for, but the silk screen clearly shows a spot for it.

![03_ozone_td07](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td07.webp)

![03_ozone_td08](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/03_ozone/td08.webp)


## BTF Light SP530E LED Controller

![04_btf_controller_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256806286614116.html?gatewayAdapt=glo2usa)

This is the first of a few LED controllers I bought to see if they are compatible with [WLED](https://kno.wled.ge/) or ESPHome.
I have some [thoughts](#side-note-the-ossha-community-is-being-heard) on the state of the LED controller market, but to summarize: it's _getting better_!

The packaging is very nondescript, containing just the controller and a manual.

![04_btf_controller_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td01.webp)

![04_btf_controller_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td02.webp)


They did include a screwdriver, which... makes me wonder how many people are buying the individual components to DIY a LED lighting solution but also don't already have a screwdriver?

![04_btf_controller_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td03.webp)


The PCB is pretty simple and, thankfully, is nothing more than an ESP32 and a level shifter.
It also includes a small condenser microphone for the music reactive mode.

![04_btf_controller_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td04.webp)


The MOSFETs look robust enough to handle a decent amount of current, and there's a decent amount of via stitching around the power traces, so it _should_ be able to handle the heat.

![04_btf_controller_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td05.webp)


![They labeled the programming header. ❤️](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/04_btf_controller/td06.webp)

_They labeled the programming header. ❤️_


### Notable ICs

[ESP32-C3](https://www.lcsc.com/product-detail/Microcontrollers-MCU-MPU-SOC_Espressif-Systems-ESP32-C3FN4_C2848860.html)

[TM(Shenzhen Titan Micro Elec) TM74HC245](https://www.lcsc.com/product-detail/74-Series_TM-Shenzhen-Titan-Micro-Elec-TM74HC245_C84121.html)

## Gledepto GL-C-015WL-D LED Controller

![05_gledepto_controller_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807239400011.html?gatewayAdapt=glo2usa)

As with the other device, this controller also has bare-bones packaging.

No screwdriver needed because all the terminals have their own locking lever!

![05_gledepto_controller_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td01.webp)


Happy to see an ESP32 module in here... on its own PCB? Complete with a micro USB port for flashing?!?

![05_gledepto_controller_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td02.webp)


Yep, the ESP32 is on a separate PCB that's connected to the main board via a header.
The silk screen on the PCB makes me think that this was _initially_ developed for an ESP8266 but then they added the header for the ESP32 module late(?) in design.

![05_gledepto_controller_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td03.webp)


The silk screen is really well done; everything is labeled well.

![Really happy to see that the GPIOs are labeled... for both the ESP32 and 8266 variant!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td04.webp)

_Really happy to see that the GPIOs are labeled... for both the ESP32 and 8266 variant!_


Because this is an "addressable" LED controller, there's not much to look at.
The GPIO from the ESP goes more or less directly to the output pins.

![05_gledepto_controller_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td05.webp)


And in case there was any doubt, the back of the PCB makes it super clear that this PCB _can_ be used with an ESP8266 or ESP32.

![05_gledepto_controller_td06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/05_gledepto_controller/td06.webp)


## SMLight A1-SLWF-03 LED Controller

![06_smlight_controller_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/06_smlight_controller/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256805957696665.html?gatewayAdapt=glo2usa)

I saved the _best_ for last!
This LED controller _really_ did surprise me with how open it is!

They're not putting a generic "ESP32 powered" message on the label leaving the reader to read between the lines / infer that the device can be flashed with alternate firmware.

They're straight up saying so!
The label has the GPIO pins clearly labeled.
It uses USB-C for power and flashing.
The USB port is _modern_ unlike [other](#generic-ozone-generator-xd001) devices I've [shown](#gledepto-gl-c-015wl-d-led-controller) in this post.
The USB port can also be used for programming and isn't hidden!

Here's to hoping that they release a version for analog LED strips! 🤞

![06_smlight_controller_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/06_smlight_controller/td01.webp)

![06_smlight_controller_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/06_smlight_controller/td02.webp)

![🇺🇦](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/06_smlight_controller/td03.webp)

_🇺🇦_


### side note: the OSS/HA community is being heard

Over the past few years, I've been scouring AliExpress to find the latest and greatest in LED controllers and every year I've been disappointed.
But over the last year or so, I've noticed a _huge_ uptick in the number of controllers that are _actually_ compatible with Home Assistant and other open-source home automation systems.

In the beginning, they were all ESP8266 based and trivial to reverse engineer and flash.
Then they started using TuYa intermediate MCUs which were a bit more tedious to reverse engineer but still possible if you were willing to put in the effort.

Now, they're using ESP32 based chips and going out of their way to LABEL which GPIOs go to which pin(s) and to generally make it TRIVIAL to flash WLED or ESPHome or whatever you want.

This means some savvy manufacturers are paying attention and recognizing the growing hobbyist market for these devices and are catering to them.
There's somebody out there that's not going to force their garbage app/api on you and is going to let you do whatever you want with the device you bought.
This was UNHEARD OF just a few years ago and I'm _so_ happy to see it happening now!

![This is such a victory for the DIY/HA/OSS community!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/06_smlight_controller/promotes_programming.webp)

_This is such a victory for the DIY/HA/OSS community!_


## G3061 PD65W Mini Hot Plate

![07_hotplate_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/07_hotplate/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256806193753913.html?gatewayAdapt=glo2usa)

And the winner of the "feels most like a hand-built prototype" award goes to... this thing!

Normally I don't like it when screws are hidden under labels / stickers... but in this case, the "sticker" is a rubber foot that doesn't even "hide" the screws.

![Those lumps under the rubber mat are the screws holding the case together.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/07_hotplate/td01.webp)

_Those lumps under the rubber mat are the screws holding the case together._


Looking from the side, the construction becomes pretty clear; it's probably just using a PCB for the heating element and the stacked PCBs separated with little brass standoffs is for thermal isolation.

![07_hotplate_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/07_hotplate/td02.webp)


USB-C for power is _always_ a good sign.
Other than that, a basic [CMSemicon](https://www.mcu.com.cn/en/) IC runs the show.

The OLED screen module is for sure one of those prototype modules meant for hobbyists!

![The use of threaded inserts for nuts just screams 'prototype' to me. Nothing wrong with that, just makes me smile.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/07_hotplate/td03.webp)

_The use of threaded inserts for nuts just screams 'prototype' to me. Nothing wrong with that, just makes me smile._


As expected, the heating element is just PCB traces on an aluminum substrate PCB.
Cheap and effective!

![07_hotplate_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/07_hotplate/td04.webp)


If anybody needs the instructions for this, I scanned a copy and you can get it from [here](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/files/07_hotplate/g3601-instructions.pdf).

## Generic USB-C Dock w/ LCD

![08_usbc_lcd_dock_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256806850754746.html?gatewayAdapt=glo2usa)

USB-C "expansion" docs are a dime a dozen now.
This caught my attention because of the LCD display on the side.
What are the odds that it presents as a second display using [display port tunneling](https://www.displayport.org/displayport-over-usb-c/) via usb-alt mode?

There are no screws, just clips.

![08_usbc_lcd_dock_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td01.webp)

![08_usbc_lcd_dock_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td02.webp)

![08_usbc_lcd_dock_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td03.webp)


It looks like a pretty straightforward de-mux and breakout PCB.
The display is an isolated / standalone component that's linked to the main PCB with just 4 wires.
What are the odds that it's hanging off a USB hub?

![I'm not exactly sure why they made the USB-C "tail" detachable.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td04.webp)

_I'm not exactly sure why they made the USB-C "tail" detachable._


The silk screen markings make me think that the "internal" name for this product is `K0108-1D`.
Interesting that the `HUB` PCB has a date code almost a full _year_ after the `LCD` PCB..
Also worth noting; the instructions that came with the device link to the same shady `key123.vip` site that the [ajazz](#ajazz-akp03e-stream-deck-clone) used so there's _definitely_ some sort of relationship between the two but I'm not sure what it is.

![08_usbc_lcd_dock_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td05.webp)

![08_usbc_lcd_dock_td06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td06.webp)


Annoyingly, they lasered off the markings on the display module.
I don't have a clip for the eeprom, but I bet that there'd be at least a few interesting strings in there :D.

![I wonder what SW1 is used for...](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td07.webp)

_I wonder what SW1 is used for..._

![Nothing much on the under side of the USB-Dock PCB.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/08_usbc_lcd_dock/td08.webp)

_Nothing much on the under side of the USB-Dock PCB._


### USB details

After seeing that the LCD is just a standalone module, I wanted to see how it presented itself to the host.
As expected, it's just a USB hub with a few devices hanging off it.
The display module is almost certainly the HID device.

```shell
❯ lsusb -vvv -t > pre-dock
❯ lsusb -vvv -t > pre-dock
❯ lsusb -vvv -t > post-dock
❯ diff pre-dock post-dock
3a4,15
>     |__ Port 003: Dev 006, If 0, Class=Hub, Driver=hub/4p, 480M
>         ID 1a40:0801 Terminus Technology Inc.
>         /sys/bus/usb/devices/1-3  /dev/bus/usb/001/006
>         |__ Port 003: Dev 007, If 0, Class=Hub, Driver=hub/4p, 480M
>             ID 1a40:0801 Terminus Technology Inc.
>             /sys/bus/usb/devices/1-3.3  /dev/bus/usb/001/007
>             |__ Port 002: Dev 009, If 0, Class=Human Interface Device, Driver=usbhid, 480M
>                 ID 5548:6667
>                 /sys/bus/usb/devices/1-3.3.2  /dev/bus/usb/001/009
>         |__ Port 004: Dev 008, If 0, Class=Vendor Specific Class, Driver=r8152, 480M
>             ID 0bda:8152 Realtek Semiconductor Corp. RTL8152 Fast Ethernet Adapter
>             /sys/bus/usb/devices/1-3.4  /dev/bus/usb/001/008
```

And here's `dmesg` with the screen _not_ attached versus _attached_:

```shell
usb 1-3: new high-speed USB device number 23 using xhci_hcd
usb 1-3: New USB device found, idVendor=1a40, idProduct=0801, bcdDevice= 1.00
usb 1-3: New USB device strings: Mfr=0, Product=1, SerialNumber=0
usb 1-3: Product: USB 2.0 Hub
hub 1-3:1.0: USB hub found
hub 1-3:1.0: 4 ports detected
usb 1-3: USB disconnect, device number 23
usb 1-3-port3: attempt power cycle
usb 1-3: new high-speed USB device number 28 using xhci_hcd
usb 1-3: New USB device found, idVendor=1a40, idProduct=0801, bcdDevice= 1.00
usb 1-3: New USB device strings: Mfr=0, Product=1, SerialNumber=0
usb 1-3: Product: USB 2.0 Hub
hub 1-3:1.0: USB hub found
hub 1-3:1.0: 4 ports detected
usb 1-3.3: new high-speed USB device number 29 using xhci_hcd
usb 1-3.3: New USB device found, idVendor=1a40, idProduct=0801, bcdDevice= 1.00
usb 1-3.3: New USB device strings: Mfr=0, Product=1, SerialNumber=0
usb 1-3.3: Product: USB 2.0 Hub
hub 1-3.3:1.0: USB hub found
hub 1-3.3:1.0: 4 ports detected
usb 1-3.4: new high-speed USB device number 30 using xhci_hcd
usb 1-3.4: New USB device found, idVendor=0bda, idProduct=8152, bcdDevice=20.00
usb 1-3.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 1-3.4: Product: USB 10/100 LAN
usb 1-3.4: Manufacturer: Realtek
usb 1-3.4: SerialNumber: 00E04C360C56
r8152-cfgselector 1-3.4: reset high-speed USB device number 30 using xhci_hcd
r8152 1-3.4:1.0: skip request firmware
r8152 1-3.4:1.0 eth0: v1.12.13
r8152 1-3.4:1.0 enp5s0f1u3u4: renamed from eth0
```

And with the screen attached, there's some additional lines as another device is enumerated:

```shell
usb 1-3.3.2: new high-speed USB device number 34 using xhci_hcd
usb 1-3.3.2: New USB device found, idVendor=5548, idProduct=6667, bcdDevice= 3.00
usb 1-3.3.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 1-3.3.2: Product: 997F1
usb 1-3.3.2: Manufacturer: 997
usb 1-3.3.2: SerialNumber: 997F11193737
on usb-0000:05:00.1-3.3.2/input0
```

## Ajazz AKP03E (Stream Deck clone)

![09_AKP03E_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807320371570.html?gatewayAdapt=glo2usa)

This is a clone/knockoff of the Elgato Stream Deck products.
I was curious to see if they did anything differently than the original for the internal hardware.
Turns out, no.
It's still a basic LCD screen behind a matrix of clear buttons.

The unit can be easily separated from its stand and there are no hidden screws!

![09_AKP03E_td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td01.webp)

![09_AKP03E_td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td02.webp)


After going around the edge w/ a spudger, the internals reveal themselves.

![09_AKP03E_td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td03.webp)


The `-VxNx` suffix on the silk screen matches the silk screen seen on the [USB-C Dock](#generic-usb-c-dock-w-lcd)... which really does make me think that there's some sort of relationship between the two products.

The board with the physical knobs/buttons is just a "pass through" PCB; everything here just goes right over the FFC to the board with the screen and buttons.

![09_AKP03E_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td04.webp)


I can't find _anything_ when googling the main IC, unfortunately.

> ARX29025
>
> Q2402

The silk screen is:

> HSV293-N3-LCD-V2P1
>
> 20240620

![09_AKP03E_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td05.webp)

![09_AKP03E_td06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/09_AKP03E/td06.webp)


The software is _super_ sketchy; just trying to find a build that actually worked on macOS was a pain!

The [official site](https://www.a-jazz.com/en/search.jsp?id=422&q=03) looks and functions like it was made with a 2000s-era website builder 🤮.

With some google-fu, I found links to a [web forum](https://bbs.key123.vip/) that may not be officially related to the product, but [has some download links](https://bbs.key123.vip/forum.php?mod=viewthread&tid=1167&highlight=AKP03) to software that is supposed to work with the device, on macOS in English.

I never got a working build of the software, but the few builds that I did try all have the same architecture; a `Qt` app that hosts plugins and does all of the thinking for the device.

With a bit more digging, I think that this "Mirabox" company is the underlying OEM and the ajazz is just a white-labeled version of this product: [Stream Dock MBox 293N3](https://miraboxbuy.com/collections/stream-dock/products/streamdock-mbox-n3).

Mirabox also appears to have a [fledgling SDK](https://sdk.key123.vip/en/guide/overview.html#stream-dock-mbox-293n4) for plugins _and_ a [Python SDK](https://github.com/MiraboxSpace/Linux-StreamDock-PythonSDK) for Linux users!

### USB details

Similar architecture; USB hub with two HID devices hanging off it; probably one HID for reading the buttons and another for writing to the screen.

```shell
❯ lsusb -vvv -t > pre-deck
❯ lsusb -vvv -t > post-deck
❯ diff pre-deck post-deck
3a4,9
>     |__ Port 003: Dev 019, If 0, Class=Human Interface Device, Driver=usbhid, 480M
>         ID 0300:3002
>         /sys/bus/usb/devices/1-3  /dev/bus/usb/001/019
>     |__ Port 003: Dev 019, If 1, Class=Human Interface Device, Driver=usbhid, 480M
>         ID 0300:3002
>         /sys/bus/usb/devices/1-3  /dev/bus/usb/001/019
```

And `dmesg`:

```shell
usb 1-3: new high-speed USB device number 22 using xhci_hcd
usb 1-3: New USB device found, idVendor=0300, idProduct=3002, bcdDevice= 0.02
usb 1-3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 1-3: Product: HOTSPOTEKUSB HID DEMO
usb 1-3: Manufacturer: HOTSPOTEKUSB
usb 1-3: SerialNumber: 0300D0781A4A
on usb-0000:05:00.1-3/input0
input: HOTSPOTEKUSB HOTSPOTEKUSB HID DEMO as /devices/pci0000:00/0000:00:01.2/0000:02:00.0/0000:03:08.0/0000:05:00.1/usb1/1-3/1-3:1.1/0003:0300:3002.00DE/input/input180
on usb-0000:05:00.1-3/input1
```

## Generic WiFI LED Wall Clock

![10_wifi_led_clock_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807326722207.html?gatewayAdapt=glo2usa)

I've been looking for a "Works with Home Assistant" version of the [Echo Wall Clock](https://www.amazon.com/Introducing-Echo-Wall-Clock---An-Echo-companion-to-see-timers-at-a-glance./dp/B07FQDMKFT) ever since Amazon discontinued it.

I was hoping this would be a simple ESP8266 or ESP32 module linked directly to the LEDs... but there's an anonymous intermediate microcontroller in there.

Assuming that the microcontroller can be removed easily and the various segments of addressable LEDs can be driven directly from an ESP8266 or ESP32, this could still be a viable base for a custom LED wall clock.

![Why bother with a custom USB cable? Just use a standard USB-C jack for power!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td01.webp)

_Why bother with a custom USB cable? Just use a standard USB-C jack for power!_

![Lots of SMD neopixels and a partition/mask system to keep high contrast between the segments.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td02.webp)

_Lots of SMD neopixels and a partition/mask system to keep high contrast between the segments._


![Kinda interesting to see how they spread the components out to not interfere with the partitions.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td03.webp)

_Kinda interesting to see how they spread the components out to not interfere with the partitions._


![10_wifi_led_clock_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td04.webp)


![PCB marking: YC-9108-WIFI-0807, Ver 1.6, 2024-6-8](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td05.webp)

_PCB marking: YC-9108-WIFI-0807, Ver 1.6, 2024-6-8_


![Markings have been sanded off. Why bother? What's the value in that?](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td06.webp)

_Markings have been sanded off. Why bother? What's the value in that?_


![I assume this is i2c for the RTC module.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/10_wifi_led_clock/td07.webp)

_I assume this is i2c for the RTC module._


## Mirabox N4

This was actually a late addition to the list.
_Technically_ I didn't order it in 2024, but after learning about the Ajazz possibly being a Mirabox product, I wanted to see if there was anything interesting in the Mirabox product line so I found this for a reasonable price and held off publishing this post until it arrived.

![11_mirabox_n4_render](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/render.webp)


[non-affiliate link](https://www.aliexpress.us/item/3256807963651419.html)

The packaging is nondescript and minimal; the unit and a nice USB A -> C cable is essentially all that's in the box.

![Did they have to use foam for this? Cardboard would have been fine...](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td01.webp)

_Did they have to use foam for this? Cardboard would have been fine..._


![The Mirabox logo on top is laser engraved which is a nice touch.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td02.webp)

_The Mirabox logo on top is laser engraved which is a nice touch._


![No screws, just clips! And they're not even that hard to find or pry up!](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td03.webp)

_No screws, just clips! And they're not even that hard to find or pry up!_


![11_mirabox_n4_td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td04.webp)


![11_mirabox_n4_td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td05.webp)


As was the case with the other LCD modules, this looks like it's just USB 2.0 module hanging off a USB hub.
Conceivably, you could embed a tiny raspberry pi zero or other _small_ computer with USB Host support inside this thing 🤔...

![Really nice ofd them to label each FPC connector.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td06.webp)

_Really nice ofd them to label each FPC connector._


![11_mirabox_n4_td07](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td07.webp)


The two internal PCBs are marked very similarly to the [USB-Dock](#generic-usb-c-dock-w-lcd) _and_ the [Stream Deck clone](#ajazz-akp03e-stream-deck-clone) which really does make me think that mirabox is the ODM for those products as well.

Markings:

> HSV293-N4-LCD
>
> 20240807
>
> HSV293-N4-SW
>
> 202400711

And the USB hub PCB in the base doesn't have a part number, just a date code.
I'd bet that they started designing this product in spring of 2024 and they finished the simplest part - the USB hub - first.

![As expected, the buttons and the lower display are really just different regions of the same LCD.](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td08.webp)

_As expected, the buttons and the lower display are really just different regions of the same LCD._


The dials are aluminum, which feels very premium for a device that's otherwise plastic.

![That plastic insert almost looks 3d printed...](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/td09.webp)

_That plastic insert almost looks 3d printed..._


![For sketchy software, it's nice to see that it's only making calls to the manufacturer's site and not 25 other ad networks...](https://karlquinsland.com/aliexpress-11-11-sale-teardowns-2024/images/11_mirabox_n4/ls_network.webp)

_Interesting to see what network calls were made..._


