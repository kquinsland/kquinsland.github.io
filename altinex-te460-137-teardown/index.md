# Altinex TE460-137 Teardown


# Altinex TE460-137 Teardown

It is surprisingly hard to find a device that can dump details about the various protocols and negotiated standards for a HDMI connection.
For how ubiquitous HDMI is, you'd _think_ that there would be a lot of devices that can do this.
There are LOADS of devices that can snoop USB and Ethernet, but HDMI is a bit of a different beast, apparently.

After a bit of searching, I found the [Altinex TE460-137](https://www.altinex.com/product/te460-137/) could do exactly what I wanted.

({{< relref "posts/2020/10/Enhanced HomeAssistantSwitchPlate" >}}) project.

Like the [HDMI CEC dongle]({{< relref "posts/2023/12/pulse-eight-hdmi-cec-injector-teardown" >}}) this device is a bit specialized and so there's not a lot of readily available information about it online.
I was curious about what's inside and how it works, so I decided to take a look.

{{< figure name="feat-td07" >}}

## Magnuson-Moss Warranty Act

{{< figure name="td06" >}}

Notice that pierced `Warranty void if removed` sticker over the bottom right screw?
Yeah, that's [**illegal**](https://www.npr.org/sections/thetwo-way/2018/04/11/601582169/warranty-void-if-removed-as-it-turns-out-feds-say-those-warnings-are-illegal).
Naturally, companies don't want you to know that the sticker is [unenforceable](https://www.ifixit.com/News/15464/warranty-voiding-stickers-are-illegal-but-these-companies-are-still-using-them).

Consider this a small [PSA](https://www.ifixit.com/News/11748/warranty-stickers-are-illegal): if you see a sticker like this, you can safely ignore it.
The company _cannot_ void your warranty for removing it but they absolutely can make your life difficult if you do.
If you get pushback, direct them to the [FTC's official statement](https://www.ftc.gov/business-guidance/resources/businesspersons-guide-federal-warranty-law) on the matter.

Anyways.

## Quick peek Inside

After removing the four screws, the case splits open easily enough.
The construction is a bit more elaborate than expected; there's a main PCB, a control PCB, and a battery interface PCB.

I was in a hurry so the photos are "as-is" and not my best work.
I've [noted down the markings on the chips](#chips-and-other-markings) and other components for future reference.

{{< figure name="td05" >}}

If the battery ever does leak, the entire PCB assembly can be removed and replaced.
It also shouldn't be that difficult to just modify the battery interface PCB to use a different battery.
The thermistor is a nice touch!

{{< figure name="td04" >}}

Taking a quick look at the control PCB, we can see a few big ICs...
The populated JTAG header is an interesting find.

{{< figure name="td03" >}}

There's not much on the rear of the control PCB:

{{< figure name="td02" >}}

Nor is there much on the side of the PCB just behind the OLED screen / front panel.
Oddly enough, each button gets _two_ sets of contacts...

{{< figure name="td01" >}}

And there's nothing interesting on the 'internal' side of the front panel:

{{< figure name="td08" >}}

## Some Technical Details

If you catch the device early enough in boot, you can see the panasonic HDMI switch chip enumerating in it's default state.
Don't ask me where the 2% overscan comes from...

{{< figure name="default-edid-enumeration" >}}

### Chips and other markings

#### Control PCB

- The main application processor is the [NXP LPC1830FET100](https://www.nxp.com/part/LPC1830FET100#/).
- It seems like all audio is handled by the [Analog Devices ADAU1701](https://www.analog.com/en/products/adau1701.html).

Oddly enough there are _two_ flash chips here. Presumably firmware storage for the  NXP and Analog Devices chips.

- [Winbond 25Q32JV](https://www.winbond.com/hq/product/code-storage-flash-memory/serial-nor-flash/?__locale=en&partNo=W25Q32JV)
- [Microchip 24LC128I](https://www.microchip.com/en-us/product/24lc128)

I didn't bother dumping either, but Altinex does make an older version of the firmware [available](#firmware).

### Main PCB

The big QFP144 chip on the main PCB is a [Panasonic MN864788](https://panasonic.encompass.com/item/10292549/Panasonic/MN864788/).
I can't find a data sheet, but one [supplier has labeled it](https://stesys.eu/ocart/index.php?route=product/product&manufacturer_id=135&product_id=2544&limit=75) a "HDMI 2.0 Receiver/Switch".
Given the proximity to the two HDMI ports ... this makes sense.

### `dmesg`

For thoroughness, here's how the device enumerates over USB when plugged in when configured in RS-232 mode:

```shell
usb 5-4.1.2.2: new full-speed USB device number 91 using xhci_hcd
usb 5-4.1.2.2: New USB device found, idVendor=04e2, idProduct=1411, bcdDevice= 0.01
usb 5-4.1.2.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 5-4.1.2.2: Product: XR21B1411
usb 5-4.1.2.2: Manufacturer: Exar Corp.
usb 5-4.1.2.2: SerialNumber: Q9065270561
usbcore: registered new interface driver xr_serial
usbserial: USB Serial support registered for xr_serial
xr_serial 5-4.1.2.2:1.0: xr_serial converter detected
usb 5-4.1.2.2: xr_serial converter now attached to ttyUSB0
```

And when in FW update mode:

```shell
usb 5-4.1.2.2: new high-speed USB device number 92 using xhci_hcd
usb 5-4.1.2.2: config 1 interface 0 altsetting 0 bulk endpoint 0x83 has invalid maxpacket 64
usb 5-4.1.2.2: config 1 interface 0 altsetting 0 bulk endpoint 0x2 has invalid maxpacket 64
usb 5-4.1.2.2: New USB device found, idVendor=1fc9, idProduct=8001, bcdDevice= 1.00
usb 5-4.1.2.2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
usb 5-4.1.2.2: Product: USB Mass Storage            
usb 5-4.1.2.2: Manufacturer: USB
usb-storage 5-4.1.2.2:1.0: USB Mass Storage device detected
scsi host4: usb-storage 5-4.1.2.2:1.0
scsi 4:0:0:0: Direct-Access     NXP      Dataflash Disk   0.00 PQ: 0 ANSI: 0
sd 4:0:0:0: [sda] 3968 512-byte logical blocks: (2.03 MB/1.94 MiB)
sd 4:0:0:0: [sda] Write Protect is off
sd 4:0:0:0: [sda] Mode Sense: 00 00 00 00
sd 4:0:0:0: [sda] Asking for cache data failed
sd 4:0:0:0: [sda] Assuming drive cache: write through
usb 5-4.1.2.2: reset high-speed USB device number 92 using xhci_hcd
usb 5-4.1.2.2: device descriptor read/64, error -110
```

### Firmware

{{< admonition note >}}

The device ships with FW 3.06A/R3.

Their [website](https://www.altinex.com/download/te460-137-firmware-update-v-3-04a-easy-mode/) only have 3.04 available.

{{< /admonition >}}

