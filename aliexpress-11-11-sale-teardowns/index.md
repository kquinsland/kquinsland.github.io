# AliExpress 11.11 Sale Teardowns

It's a whole Grab bag of worth of #TwoMinuteTeardowns.

Canonical: https://karlquinsland.com/aliexpress-11-11-sale-teardowns/
Published: 2023-12-01
Updated: 2023-12-01
Tags: two-minute-teardown


---

<!-- markdownlint-disable-file MD024 -->
# 11-11 day teardowns

[Single's Day](https://en.wikipedia.org/wiki/Singles%27_Day) is a big deal in China and AliExpress has a big sale in celebration.

In addition to project supplies, I found quite a few items that can only be classified as  "ohh, that looks interesting and I need another $11 in the cart to unlock free shipping..." items.

I was planing on doing a series of posts covering some of the stuff I bought but nothing on it's own was really "worth" a full post so instead of a bunch of short ["Two Minute Teardown"](/tags/two-minute-teardown/) posts, I decided to just roll everything into this massive post.

And with that, here's a whole grab bag of worth of #TwoMinuteTeardowns with the appropriate level of detail for each item.

## Super cheap 'AirTag'

Product in question: [Mini Tracker Bluetooth4.0 Smart Locator Smart Anti Lost Device Locator Mobile Keys Pet Kids Finder For Apple](https://www.aliexpress.us/item/3256804801453934.html?gatewayAdapt=glo2usa)

This was a mistake on my part.
I saw a BTLE powered device tracker shaped and styled exactly like an air tag for **$5 shipped** and thought "why not?"
One good reason why not: it's not compatible with Apple's Find My network.

In hindsight, if it was only $5 and if it worked 'natively' with apple devices... I'd have heard about it before browsing Ali Express!

![air-tag-td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-tag/td02.webp)


![air-tag-td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-tag/td01.webp)


### Markings and other details

The main CPU appears to be a [Lenze Tech ST17H66B](https://www.lenzetech.com/public/store/pdf/jsggs/ST17H66B2_BLE_SoC_Datasheet_v1.1.2.pdf)

The instructions point to an [`iSearching` app](https://www.qrtransfer.com/iSearching.html)
I didn't bother installing it, here's what a quick scan of the BTLE characteristics shows quite a few services and characteristics:

![air-tag-td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-tag/td03.webp)


## TuYa WiFi Power Monitor

Product in question: [16A Tuya WIFI Smart Socket AC220V 110V Digital Wattmeter EU Plug Electricity consumption Power KWH US AU FR Power Energy Meter](https://s.click.aliexpress.com/e/_mNLeK44)

![tuya-power-meter-01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/tuya-power-meter/td01.webp)


![tuya-power-meter-02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/tuya-power-meter/td02.webp)


TuYa module doesn't seem to be running the show; most of the pads are not connected; only the UART pins seem to be going somewhere.

![tuya-power-meter-03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/tuya-power-meter/td03.webp)


A bit more going on than you'd expect at first; I was expecting a simple Switch Mode power supply driver for DC and then a single MCU to do everything but the WiFi.
Not so!
There's a dedicated chip for power monitoring, BT, Power supply, and the main MCU...

![tuya-power-meter-04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/tuya-power-meter/td04.webp)


![tuya-power-meter-05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/tuya-power-meter/td05.webp)


### Markings and other details

- [KP3211BSG](https://en.sekorm.com/doc/2232723.html)
- [CH573F](https://www.wch-ic.com/products/CH573.html)
- [GMT024-01](https://www.aliexpress.us/item/2255799832871961.html?gatewayAdapt=glo2usa4itemAdapt)
- [TuYa CBS3 module](https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076)
- [BL0942](https://www.lcsc.com/product-detail/Energy-Metering-ICs_BL-Shanghai-Belling-BL0942_C2837510.html)

- `BP0S229-28A2`: This is the same chip that shows up in the [pet RFID reader](#pet-rfid-tag-reader) and I cant' find an exact match for it.

- LCD is marked `AVDD AVCL` which seems to be pretty generic

## TuYa WiFi Air Quality Monitor

Product in question: [Tuya Wifi Portable Air Quality Meter 8in1 PM1.0 PM2.5 PM10 CO2 TVOC HCHO Temperature and Humidity Tester Carbon Dioxide Detector](https://s.click.aliexpress.com/e/_mPXVCDO)

Unfortunately you have to peel off the screen cover to get access to the screws.
At least they're just regular philips screws and not some weird security screws.

Interesting to see the IR sensor at the top. Presumably this is for proximity sensing to turn the screen on and off and not some kind of IR blaster or remote control.

![air-mon-td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-mon/td01.webp)


Makes sense that the various sensors would be facing the vents holes.

![air-mon-td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-mon/td02.webp)


Looks like it's using a micro controller to do the heavy lifting and only communicate to the TuYa module via UART.
This is a common and well-supported pattern for these kinds of devices ... even if it does mean that it won't be _trivial_ to just replace the TuYa module with an ESPHome or Tasmota compatible module.

Sniffing the UART shouldn't be that difficult but it's not something I'm going to do in this post.

![air-mon-td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-mon/td03.webp)


![air-mon-td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-mon/td04.webp)


The various sensors aren't easily removed or clearly identified so your guess is as good as mine.
The main PCB has a few unpopulated connections so this module either works with a different suite of sensors or - more likely - supports a few different footprints for each sensor.

![air-mon-td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/air-mon/td05.webp)


### Markings and other details

- Main MCU: FMD B2CACSK. I can't find _anything_ on this chip.
- TuYa module: [CBU](https://developer.tuya.com/en/docs/iot/cbu-module-datasheet?id=Ka07pykl5dk4u)

PCB is marked with:

```text
    PV28_TFT_v3
    2022/10/25
```

Screen is marked with:

```text
    LY028S24256
```

## WaveShare PoE RS232/485/422 adapter

Product in question: [with POE optional Modbus MQTT JSON serial server RS485 RS232 RS422 to Ethernet TCP/IP to serial converter](https://s.click.aliexpress.com/e/_mPR1X7e).

![poe-rs232-td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/poe-rs232/td01.webp)


[Line termination resistors](https://electronics.stackexchange.com/questions/413295/rs485-termination) that can be switched  in/out via jumpers is a very nice touch!

![poe-rs232-td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/poe-rs232/td02.webp)


Neat and clean PCB layout with adequate protections on the RS485/422/232 lines.

![poe-rs232-td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/poe-rs232/td03.webp)


Unfortunately the main MCU has it's markings lasered off so your guess is as good as mine.

![poe-rs232-td04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/poe-rs232/td04.webp)


![poe-rs232-td05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/poe-rs232/td05.webp)


### Markings and other details

All the RS485/422/232 seems to be done with Sipex/MaxLinear chips:

- RS484/422 via [MaxLinear SP483E](https://assets.maxlinear.com/web/documents/sipex/datasheets/sp483e.pdf)

- RS232 via [Max Linear (Sipex) SP3222E](https://assets.maxlinear.com/web/documents/sipex/datasheets/sp3222e_sp3232e.pdf).

- Isolation between PoE and Comms via [2PAI SEMI 141E61](https://www.aliexpress.us/item/3256804720041549.html?gatewayAdapt=glo2usa4itemAdapt)

There's also a few Diodes Inc. chips on the board:

- PoE rectification via [Diodes Incorporated MB10F](https://www.diodes.com/assets/Datasheets/MB10F.pdf)

- PoE power switching via [Diodes Incorporated SBR8U60P5](https://www.diodes.com/assets/Datasheets/SBR8U60P5.pdf)

PoE negotiation via [Silan SD4950](https://www.silan.com.cn/en/index.php/product/details/3145.html)

## Asometech WLX-X69 USB-C power brick

Product in question: [140W 6 Ports PD Fast Charger 30W Multi USB-C Fast Charging Station with LED Display for IPhone 14 13 Pro Max Samsung Xiaomi lpad](https://s.click.aliexpress.com/e/_mPlQZcs)

![usbc-power-01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td01.webp)


The mirror finish ... is certainly a choice.
It gets better once you remove the protective film, at least.

![usbc-power-02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td02.webp)


There are no screws holding the case together; just pry the back panel off, slide out the power supply and you can access the front PCB that does the actual USB-C power negotiations.

![usbc-power-03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td03.webp)


![usbc-power-04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td04.webp)


![usbc-power-05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td05.webp)


![usbc-power-06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td06.webp)


I am 100% sure that this power supply was originally designed for something else.
One of the original leads was cut off before the power supply was re-purposed.

![usbc-power-07](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td07.webp)


After removing the power distribution board from the case, it's quite clean!
It would also be trivial to use this board in other projects 🤔.

![usbc-power-08](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td08.webp)


Single CPU to run the show and a USB-C power negotiation chip per port.

![usbc-power-09](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td09.webp)


![usbc-power-10](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td010.webp)


Too much glue keeping everything the power supply together for me to bother taking it apart further.
It's got some heft to it but I'm not able to easily check out quality of transformer or other components.

![usbc-power-11](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/usbc-power/td011.webp)


### Markings and other details

- Main CPU is marked `FMD A3JMLTH` but I can't find any information on it.
- USB-C PD negotiations handled with [ZHUHAI ISMARTWARE TECHNOLOGY SW3526](https://www.ismartware.com/upload/admin/20210312/202103121815173213.pdf)

## Pet RFID tag reader

Product in question: [134.2Khz RFID Animal Tag Reader Handheld Pet Microchip Scanner IISO11784/85/FDX-B/EMID Data Storage Microchip For Pet/Dog/Cat/Pig](https://s.click.aliexpress.com/e/_mrfMB5i)

![pet-rfid-td01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/pet-rfid/td01.webp)


Not much on the back of the PCB: just a beeper and generic battery management IC.

![pet-rfid-td02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/pet-rfid/td02.webp)


I honestly thought this would be more integrated.
Instead there's quite a few chips on the board.

![pet-rfid-td03](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/pet-rfid/td03.webp)


### Markings and other details

- [Microchip MCP602](https://www.microchip.com/en-us/product/mcp602): OpAmp, likely used for processing the signal from the antenna.

- [Panchip XN297L8W](https://datasheetspdf.com/datasheet/XN297.html): 2.4 ghz radio

- `MT00606XA`: I can't find any information on this chip.

- `BP0S229-28A2`: A chip from [Zhuhai Jieli Technology](https://www.zh-jieli.com/) that I can't seem to 100% identify.
The markings match the same chip that showed up in the [power monitor](#tuya-wifi-power-monitor) above.
The power monitor has both WiFi and Bluetooth so it's not a stretch to think that this chip is Bluetooth connectivity.

PCB is marked `W90A-V1.9`

## ATK-PTDP100 DC Power Supply

Product in question: [DP100 DC Power Supply Adjustable Digital DC Power Supply MINI Portable Lab Source Power Supply Voltage Regulator Switch 30V 5A](https://s.click.aliexpress.com/e/_mOfFkQY)

This is nothing more than a desk-friendly version of the [RD6006](/electronics-lab-enhanced-bench-psu/) power supply and it's amazing.

Here's everything that came in the box:

![dp100-02](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td02.webp)


It shipped with this power supply and a barrel jack to USB-C adapter... because the DP100 actually expects its power to come _from_ a USB-C/PD capable source.
It'll be trivial to use this power supply with a USB-C power bank.
It - annoyingly - has a USB-A **_male_** port for data logging / firmware updates.
At least they included a Male <-> Male USB A cable...
I would have preferred two USB-C ports with clear labeling about which was meant for PD input and which was meant for PC communications.

![dp100-05](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td05.webp)


For reference, the power supply is pretty beefy:

![dp100-01](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td01.webp)


Terminals are compact but decently high quality.
The little red and black rubber bands are a nice touch.

![dp100-04](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td04.webp)


Annoying security torx screws under the rubber feet.

![dp100-07](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td07.webp)


Screen is a bit cramped and navigation is a bit clunky but once you get used to it, it's not bad.

![dp100-06](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td06.webp)


After removing the screws and prying the case apart, it's quite clean inside.

![dp100-08](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td08.webp)


Main CPU next to some unpopulated pin headers and some loctite screwing the PCB to the terminals.

![dp100-12](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td012.webp)


On the other side of the PCB is a metal can with all the fun power conversion stuff.

![dp100-15](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td015.webp)


They're doing a [4 wire measurement](https://www.allaboutcircuits.com/textbook/direct-current/chpt-8/kelvin-resistance-measurement/) at the terminals via a small secondary PCB.

![dp100-17](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td017.webp)


And after removing the can, it's just a bunch of filter caps and an inductor.

![dp100-18](https://karlquinsland.com/aliexpress-11-11-sale-teardowns/images/dp100/td018.webp)


### Markings and other details

Unfortunately most of the ICs didn't photograph well or otherwise have scratched / obscured markings.
Most of the power switching ICs _look_ like they're from Monolithic Power Systems but I can't confirm exactly.

[WCH(Jiangsu Qin Heng) CH224K](https://www.lcsc.com/product-detail/USB-ICs_WCH-Jiangsu-Qin-Heng-CH224K_C970725.html)

Main CPU is [Artery AT32F415](https://www.arterychip.com/en/product/AT32F415.jsp)

