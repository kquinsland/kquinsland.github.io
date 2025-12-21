# #TwoMinuteTeardown: Ubiquiti SFP Wizard

# Two Minute Teardown: Ubiquiti SFP Wizard

{{<figure name="feat-render">}}

SFP modules interface to their host over what is essentially a low speed i2c interface and a few high-speed differential pairs.

Devices that can interrogate and re-program the SFP modules have been around for a while but they are either niche, esoteric, expensive, or all of the above.

People have been [hacking on these things](https://forums.servethehome.com/index.php?threads/transceiver-password-collection.39458/) for a while:

- [Writing protected SFP / QSFP / XFP and searching password (brute force method) - REVELTRONICS](https://forum.reveltronics.com/viewtopic.php?t=527)
- [GitHub - hfuller/transceiver-notes: Notes about hacking, programming, and otherwise dinking with SFPs and other optical transceivers.](https://github.com/hfuller/transceiver-notes)
- [GitHub - SloMusti/sfpddm: Arduino library for interfacing SFP modules and reading DDM information as per SFF-8472](https://github.com/SloMusti/sfpddm)
- [Read SFP I2C via CH341a Programmer &#8211; HITOHA.もえ](https://www.hitoha.moe/read-sfp-i2c-via-ch341a-programmer)
- [sfp-experimenter](https://osmocom.org/projects/misc-hardware/wiki/Sfp-experimenter)

The [Ubiquiti SFP Wizard](https://store.ui.com/us/en/category/accessories-modules-fiber/collections/accessories-pro-direct-attach-cables/products/uacc-sfp-wizard) is a new entrant to the market.

This is not a review.
The packaging is plain recycled (?) cardboard with white ink; it doesn't photograph well so this is also not an unboxing.

## Teardown

This thing is a massive pain in the ass to open.
I swear, somebody on the design team must be an investor in a glue factory.

The device is made up of an inner frame which holds the electronics.
Two "structural" caps are attached to the midframe with glue ans screws.
The outer case / panels snap to this midframe and there are two decorative "trim" caps that complete the exterior.

Virtually everything has at least _some_ glue on it.

Anyways, start with the trim cap on the SFP module side.
You will then want to begin gently trying to pry the back half of the case away from the front half.
Once the clips along the side are undone, there's one more clip that holds the back panel to the midframe.

You'll need a bright light and a long-neck screwdriver to hit it, but if you look through the gap between the two sfp module bays you should be able to see the tab.

{{<figure name="td-01">}}

Instead of weights to make the thing feel more premium in the hand, how about a bigger battery that's also user-replaceable!?

The LCD is heat-staked into the front panel.

{{<figure name="td-02">}}

Yep, it's an ESP32. Not expected ... but welcome!

{{<figure name="td-03">}}

There's a few more interesting things on the underside of the board... including a few headers that are almost certainly for programming/debugging.

{{<figure name="td-04">}}

That's about it! Once you manage to get in, it's a pretty simple device.

## misc

As soon as I saw the ESP32, I had to see if the USB-C port was connected to it (or just for power).
Yep, it's wired up as a USB device:

```shell
[123456.747577] usb 5-4.1.4: new full-speed USB device number 61 using xhci_hcd
[123456.854812] usb 5-4.1.4: New USB device found, idVendor=303a, idProduct=1001, bcdDevice= 1.01
[123456.854818] usb 5-4.1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[123456.854820] usb 5-4.1.4: Product: USB JTAG/serial debug unit
[123456.854822] usb 5-4.1.4: Manufacturer: Espressif
[123456.854824] usb 5-4.1.4: SerialNumber: DE:AD:BE:EF:AA:BB
[123456.902886] cdc_acm 5-4.1.4:1.0: ttyACM0: USB ACM device
[123456.902918] usbcore: registered new interface driver cdc_acm
[123456.902920] cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
```

I wasn't able to use _just_ `esptool` to reboot into bootloader mode and I didn't look for `boot0` pins so no attempt to dump the firmware was made.
If they didn't trigger any of the eFuse features, this thing could end up being a nice little SFP programmer for hobbyists that want to hack on SFPs beyond what the Ubiquiti stock firmware allows.

{{< admonition warning >}}
Do not update firmware above 1.1.0. Apparently the device can [no-longer flash _arbitrary__ modules after updating](https://community.ui.com/releases/SFP-Wizard-1-1-1/3249a171-c563-4fe1-91de-6b670057b541).

It's not clear if that's a mistake or they caved to some pressure from the SFP module manufacturers.
{{< /admonition >}}

### Other ICs

- [Winbond 25Q256JVFQ](https://www.winbond.com/hq/product/code-storage-flash-memory/serial-nor-flash/?__locale=en&partNo=W25Q256JV). This is almost certainly for storing all the SFP module "profiles" that the device supports.

- There's a TI part labeled `PD534 2CKG4 A49D` which is likely a [PCA9534A](https://www.ti.com/lit/ds/symlink/pca9534.pdf) which is an IO expander. I have absolutely no idea what it's being used for in this design... the ESP32 has plenty of GPIOs available.

