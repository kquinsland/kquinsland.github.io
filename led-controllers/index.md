# #TwoMinuteTeardown: Multiple LED Controllers

<!-- markdownlint-disable MD024 -->
## Introduction

Bit of a change in format; instead of one _massive_ post for all the misc. 11-11 teardowns, I'm grouping them.
This post is the "LED Controller" group.
I managed to snag the following on Ali Express:

- [Gledepto GL-C-618WL LED Controller](https://www.aliexpress.us/item/3256810107702305.html)
- [IoTorero LS11PDual LED Controller](https://www.aliexpress.us/item/3256810030310796.html)
- [SMLIGHT-A1-SLWF-09 LED Controller](https://www.aliexpress.us/item/3256809568993977.html)

To continue from [last year's trend]({{< relref "/posts/2024/12/aliexpress-11-11-sale-teardowns-2024" >}}#side-note-the-ossha-community-is-being-heard), I waned to celebrate that all three of these LED controllers have:

- [WLED](https://kno.wled.ge/) firmware flashed from the factory
- GPIO labeled for hacking
- No firmware lock-in; the USB interface shows up as a standard serial device on Linux so you don't even need to open it up to re-flash it!

The broad changes compared to last year are:

- Designs are more modular, allowing for upgrades to be added to a flexible base design
- Ethernet is a common upgrade option now for more reliable LAN control
- PoE is an option on some models for easier deployment in locations without easy access to power outlets

Beyond that, the internals are starting to converge on similar designs with similar components.
I have created an `ICs and Markings` section for each teardown to help identify the key components used in each design ... but you'll see that the same chips keep showing up across multiple designs.

## Gledepto GL-C-618WL LED Controller

{{<figure name="feat-gledepto-render">}}

Each connector is labeled with the GPIO that drives it <3.
Chunky user replicable fuse with a label stating the form factor and rating <3.

Not pictured; the back of the device has no screws.
A simple pry tool is all you need to open it up.

{{<figure name="gledepto_td01">}}

About what you'd expect from a WLED controller. Layout seems reasonable and intuitive.

{{<figure name="gledepto_td02">}}

{{<figure name="gledepto_td03">}}

{{<figure name="gledepto_td04">}}

Not exactly sure why they have MOSFETS immediately after the power input _and_ for the LED outputs.
{{<figure name="gledepto_td05">}}

It's not marked on the box or enclosure or mentioned in the product listing but it looks like you can select between termination resistors (24.9/33Ω) via switch.
This is a nice touch for reliability over long runs of LED strip ... but I'm not sure how many users will actually need to use it with that 74HC chip there.

No other controller I've seen has this feature so far so I appreciate the attention to detail here.

{{<figure name="gledepto_td06">}}

Beyond that, nice of them to mark what type of MIC is used and the GPIOs it's wired to.

{{<figure name="gledepto_td07">}}

I would expect the fuse to be the over-current protection for the entire board.... so I'm not sure why there's a pair of MOSFETs right after the DC input.

{{<figure name="gledepto_td08">}}

### Misc

#### ICs and Markings

- [`All Power G050N06K`](https://all-power.com/module/product/manual_download.php?b_idx=1520&idx=719):
- [`All Power 126P03K`](https://all-power.com/module/product/manual_download.php?b_idx=868&idx=501)
- [`SMSC 8720A`](https://ww1.microchip.com/downloads/en/devicedoc/8720a.pdf): Ethernet PHY
- [`CH340K`](https://www.lcsc.com/datasheet/C968586.pdf): Handles the USB to UART conversion
- [`Nexperia 74HCT125D`](https://assets.nexperia.com/documents/data-sheet/74HC_HCT125.pdf)

#### `dmesg`

When plugged in via USB, shows up as a CH341 device:

```shell
# dmesg
[123456.814962] usb 5-4.1.4: new full-speed USB device number 77 using xhci_hcd
[123456.914683] usb 5-4.1.4: New USB device found, idVendor=1a86, idProduct=7522, bcdDevice= 2.64
[123456.914688] usb 5-4.1.4: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[123456.914690] usb 5-4.1.4: Product: USB Serial
[123456.986923] usbcore: registered new interface driver ch341
[123456.986940] usbserial: USB Serial support registered for ch341-uart
[123456.986960] ch341 5-4.1.4:1.0: ch341-uart converter detected
[123456.999771] usb 5-4.1.4: ch341-uart converter now attached to ttyUSB0
```

I did not bother trying to check for logging output over the serial interface since WLED is already installed and there's no mystery firmware to reverse engineer.

## IoTorero LS11PDual LED Controller

{{<figure name="iotorero-render">}}

{{< admonition note >}}

IoTorero appears to be a sub-brand of [athom](https://www.athom.tech/); the designs are very similar.

{{< /admonition >}}

Again, nothing to unscrew on the back; just a pry tool to open it up.
The tracks are quite wide and the extra solder implies some beefy current handling.

{{<figure name="iotorero_td01">}}

This design isn't quite so modular or as sophisticated so it's easier to follow.
The individual GPIOs for the LED channels are labeled on the silkscreen but the integrated button and microphone are not so this is slightly less friendly for hacking.

{{<figure name="iotorero_td02">}}

{{<figure name="iotorero_td03">}}

### Misc

#### ICs and Markings

Some of these should look familiar if you've read the Gledepto teardown above:

- [`SMSC 8720A`](https://ww1.microchip.com/downloads/en/devicedoc/8720a.pdf): Ethernet PHY
- [`CH340K`](https://www.lcsc.com/datasheet/C968586.pdf): Handles the USB to UART conversion
- [`Alpha and Omega D4184`](https://www.alldatasheet.com/datasheet-pdf/view/485386/AOSMD/D4184.html): N-Channel MOSFET
- `74HC245TA`, Unknown Manufacturer: Octal Bus Transceiver / level shifter

#### `dmesg`

Nothing unusual here; I did not bother trying to check for logging output over the serial interface since WLED is already installed and there's no mystery firmware to reverse engineer.

```shell
[123456.526400] usb 5-4.1.4: new full-speed USB device number 79 using xhci_hcd
[123456.626682] usb 5-4.1.4: New USB device found, idVendor=1a86, idProduct=7522, bcdDevice= 2.64
[123456.626688] usb 5-4.1.4: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[123456.626690] usb 5-4.1.4: Product: USB Serial
[123456.668754] ch341 5-4.1.4:1.0: ch341-uart converter detected
[123456.682751] usb 5-4.1.4: ch341-uart converter now attached to ttyUSB0
```

## SMLIGHT-A1-SLWF-09 LED Controller

{{<figure name="smlight-render">}}

And the best for last...
This one is _very_ modular; you can upgrade the "base" model with PoE/Ethernet and a Microphone via daughterboards.

There are a few other "nice touch!" features here too:

- USB-C power input is a nice touch too. Shame it's limited to 12v. Would have been nice to see PPS used to negotiate between 12v or 20v as needed.
- QR Codes on the silk screen and labels to help track down documentation or order parts.

{{<figure name="smlight_td01">}}

Silkscreen makes it super clear which GPIOs are used for what purpose.

{{<figure name="smlight_td02">}}

{{<figure name="smlight_td03">}}

{{<figure name="smlight_td04">}}

{{<figure name="smlight_td05">}}

{{<figure name="smlight_td06">}}

{{<figure name="smlight_td07">}}

{{<figure name="smlight_td08">}}

{{<figure name="smlight_td09">}}

{{<figure name="smlight_td10">}}

{{<figure name="smlight_td11">}}

### Misc

#### ICs and Markings

The ethernet module features the familiar `SMSC 8720A` but beyond that:

- [`APM AP90P03D`](https://datasheet4u.com/datasheets/APM/AP90N03D/1573548): MOSFET for controlling power out to the LED strips.
- [`WCH CH9102F`](https://www.lcsc.com/datasheet/C2858418.pdf): USB to UART bridge chip.
- [`Toll Semiconductor TMI7321`](https://www.toll-semi.com/PDPoEICir2/214.html): PoE controller IC.

I didn't bother lifting the metal can off of the main microcontroller but based on the other two designs here it's presumably an ESP32 variant (does WLED even support anything else?).
It's also identical in appearance to the ESP32 module used on the [SMLIGHT SLZB-MRW10
]({{< relref "/posts/2025/12/slzb-mrw10" >}}) so presumably the same module is used across multiple SMLIGHT products.

I didn't get a clear photo of the markings on the USB/PD chip either.

#### `dmesg`

Nothing unusual here; I did not bother trying to check for logging output over the serial interface since WLED is already installed and there's no mystery firmware to reverse engineer.

```shell
[123456.503226] usb 5-4.1.4: new full-speed USB device number 42 using xhci_hcd
[123456.597674] usb 5-4.1.4: New USB device found, idVendor=1a86, idProduct=55d4, bcdDevice= 4.44
[123456.597679] usb 5-4.1.4: New USB device strings: Mfr=0, Product=2, SerialNumber=3
[123456.597682] usb 5-4.1.4: Product: USB Single Serial
[123456.597684] usb 5-4.1.4: SerialNumber: 5A<...>8
[123456.647727] cdc_acm 5-4.1.4:1.0: ttyACM0: USB ACM device
```

