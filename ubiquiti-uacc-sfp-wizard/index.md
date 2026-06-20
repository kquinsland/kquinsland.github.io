# #TwoMinuteTeardown: Ubiquiti SFP Wizard

Canonical: https://karlquinsland.com/ubiquiti-uacc-sfp-wizard/
Published: 2025-12-18
Updated: 2025-12-18
Tags: two-minute-teardown


---

# Two Minute Teardown: Ubiquiti SFP Wizard

{{< admonition important "Disregard note below about 1.0.5" >}}

To make a long story short, I allocated a few hours to build a [tool that can read/write the SFP module's EEPROM](https://github.com/kquinsland/sfp-eeprom-tool).
I then wired this up to a logic analyzer and compared the read/write operations to what the SFP Wizard does when it tries to unlock a module.

TL;DR:

- I didn't see a meaningful difference in what my tool read from an EEPROM compared to the SFP Wizard.
- Version 1.0.5 is the only firmware version that does not give me a "failed, locked" error when I try to write... But it's also the only version that just keeps trying to write despite despite the fact that bytes are not changing on the EEPROM.
- The assessment [here](https://github.com/vitaminmoo/sfpw-tool/blob/main/doc/HOW_TO_DOWNGRADE_AND_WHY_NOT_TO.md) is correct; there's no value in downgrading. In fact, it's (slightly) cheaper to buy a basic USB <-> I2C adapter and use my tool (which lets you try any arbitrary password) than it is to buy a SFP Wizard. The Wizard is a hell of a lot more polished, though.

{{< /admonition >}}

{{< admonition warning "Update / Do you have a device running Firmware version 1.0.5?" >}}

Details are [below](#update-some-news-on-the-esp32-side-of-things). but I wanted to put this up at the very top for visibility.

**If you have a SFP Wizard running Firmware version 1.0.5, _DO NOT_ update it.**

**If you have a COPY of the 1.0.5 OTA file, please keep it safe and get in touch with me.**

[Some work](#did-they-cripple-the-device-on-purpose) has been done to understand precisely _what_ changed between early FW versions and the more recent FW versions that have crippled the device's ability to program arbitrary SFP modules.

{{< /admonition >}}

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
**Do not update firmware above 1.1.0.** Apparently the device can [no-longer flash _arbitrary_ modules after updating](https://community.ui.com/releases/SFP-Wizard-1-1-1/3249a171-c563-4fe1-91de-6b670057b541).

It's not clear if that's a mistake or they caved to some pressure from the SFP module manufacturers.
{{< /admonition >}}

### Other ICs

- [Winbond 25Q256JVFQ](https://www.winbond.com/hq/product/code-storage-flash-memory/serial-nor-flash/?__locale=en&partNo=W25Q256JV). This is almost certainly for storing all the SFP module "profiles" that the device supports.

- There's a TI part labeled `PD534 2CKG4 A49D` which is likely a [PCA9534A](https://www.ti.com/lit/ds/symlink/pca9534.pdf) which is an IO expander. I have absolutely no idea what it's being used for in this design... the ESP32 has plenty of GPIOs available.

## Update: some news on the ESP32 side of things

Briefly:

- We have tooling to fetch the OTA images from the source, they can be analyzed with standard reverse engineering tools.
  - Very early firmware versions (1.0.5) appear to have been removed from the server :(
- The OTA images can be flashed to the device to downgrade or upgrade the firmware
- The ESP32 has security features enabled that prevent dumping the full flash contents or modifying the bootloader/application images without bricking the device.

### Tooling

There have been _two_ attempts at reverse engineering the firmware and making this tool a bit more useful than the stock Ubiquiti firmware.

- [josiah-nelson/SFPLiberate](https://github.com/josiah-nelson/SFPLiberate): I have not tried this tool yet but it does seem to be _very_ vibe-coded and does not appear to be as accurate/technically correct as the other tool below.

- [vitaminmoo/sfpw-tool](https://github.com/vitaminmoo/sfpw-tool): this is a simpler `go` based tool that I have used and does work with my SFP Wizard.

Between the two projects, there are some _interesting_ [details about the API](https://github.com/vitaminmoo/sfpw-tool/blob/main/API.md).
After skimming, it appears that the app talks to the device over a custom-ish protocol that is _heavily_ inspired by HTTP/REST principles.
Given that this thing has an ESP32 inside, I can't help but wonder if there was ever a plan to have WiFi connectivity in this thing at some point.

### ESP32 Security

[Above](#misc), I mentioned that I was not able to dump the firmware using `esptool`.
As it turns out, this is effectively because Ubiquiti has enabled some of the ESP32's security features and locked down the chip.
It is only possible to (partially) interact with the bootROM very early on in the boot process but it _is_ possible.

From the powered off state, you have about 700ms to

1. press the power button
2. plug the USB cable in
3. run `esptool`

If you're too slow, `dmesg` will show something like this:

```shell
❯ sudo dmesg -w
[12345.472386] usb 5-4.1.3.4: new full-speed USB device number 99 using xhci_hcd
[12345.545415] usb 5-4.1.3.4: device descriptor read/64, error -32
[12345.723687] usb 5-4.1.3.4: device descriptor read/64, error -32
[12345.899399] usb 5-4.1.3.4: new full-speed USB device number 100 using xhci_hcd
[12345.973426] usb 5-4.1.3.4: device descriptor read/64, error -32
[12346.147418] usb 5-4.1.3.4: device descriptor read/64, error -32
[12346.250631] usb 5-4.1.3-port4: attempt power cycle
[12346.845427] usb 5-4.1.3.4: new full-speed USB device number 101 using xhci_hcd
[12346.845497] usb 5-4.1.3.4: Device not responding to setup address.
# <...>
[12347.746642] usb 5-4.1.3-port4: unable to enumerate USB device
```

If you do get the timing right, you will get something like this:

```shell
❯ esptool --port /dev/ttyACM0 --verbose get-security-info
esptool v5.1.0
Serial port /dev/ttyACM0:
Connecting...
Detecting chip type... ESP32-S3
Connected to ESP32-S3 on /dev/ttyACM0:
Chip type:          ESP32-S3 in Secure Download Mode

Warning: Stub flasher is not supported in Secure Download Mode, it has been disabled. Set --no-stub to suppress this warning.

Security Information:
=====================
Flags: 0x000006f5 (0b11011110101)
Key Purposes: (2, 3, 9, 0, 0, 0, 12)
  BLOCK_KEY0 - XTS_AES_256_KEY_1
  BLOCK_KEY1 - XTS_AES_256_KEY_2
  BLOCK_KEY2 - SECURE_BOOT_DIGEST0
  BLOCK_KEY3 - USER/EMPTY
  BLOCK_KEY4 - USER/EMPTY
  BLOCK_KEY5 - USER/EMPTY
Chip ID: 9
API Version: 0
Secure Boot: Enabled
Secure Boot Key Revocation Status:

        Secure Boot Key1 is Revoked

        Secure Boot Key2 is Revoked

Flash Encryption: Enabled
SPI Boot Crypt Count (SPI_BOOT_CRYPT_CNT): 0x7
Dcache in UART download mode: Disabled
Icache in UART download mode: Disabled
JTAG: Permanently Disabled

Hard resetting via RTS pin...
```

When you use the `sfpw-tool` to [pull down the exiting firmware](https://github.com/vitaminmoo/sfpw-tool?tab=readme-ov-file#firmware-management) and then analyze them, you'll see output like this:

```shell
❯ esptool image-info sfpw_v1.1.0.bin
esptool v5.1.0
Image size: 1970176 bytes
Detected image type: ESP32-S3

ESP32-S3 Image Header
=====================
Image version: 1
Entry point: 0x40375858
Segments: 6
Flash size: 32MB
Flash freq: 80m
Flash mode: DIO

ESP32-S3 Extended Image Header
==============================
WP pin: 0xee (disabled)
Flash pins drive settings: clk_drv: 0x0, q_drv: 0x0, d_drv: 0x0, cs0_drv: 0x0, hd_drv: 0x0, wp_drv: 0x0
Chip ID: 9 (ESP32-S3)
Minimal chip revision: v0.0, (legacy min_rev = 0)
Maximal chip revision: v0.99

Segments Information
====================
Segment   Length   Load addr   File offs  Memory types
-------  -------  ----------  ----------  ------------
     0  0xc0a5c  0x3c100020  0x00000018  DROM
     1  0x06c00  0x3fc9d600  0x000c0a7c  BYTE_ACCESSIBLE, MEM_INTERNAL, DRAM
     2  0x0898c  0x40374000  0x000c7684  MEM_INTERNAL, IRAM
     3  0xf4fd4  0x42000020  0x000d0018  IROM
     4  0x10c18  0x4037c98c  0x001c4ff4  MEM_INTERNAL, IRAM
     5  0x0a3b4  0x00000000  0x001d5c14  PADDING

ESP32-S3 Image Footer
=====================
Checksum: 0xd2 (valid)
Validation hash: 036b431463c9e6c363812d12727aacda8bce1da89a45a79c26ea4a3311c7ac47 (valid)

Application Information
=======================
Project name: uacc_sfp_wizard
App version: sfp_wizard-1.1.0-dirty
Compile time: Nov 13 2025 12:35:57
ELF file SHA256: 76062ef9d3fd0391282cd9fe4ef462cf096064a3b23dc507ae02d3cda422df8f
ESP-IDF: v5.3.1
Minimal eFuse block revision: 0.0
Maximal eFuse block revision: 0.0
Secure version: 0
```

Not pictured here, but `strings` on the bin file shows that it's plain-text.

This means that:

- The OTA firmware files are 'plain text' `.bin` files that can be analyzed with standard ESP32 tooling but they do not contain the entire firmware image; they are effectively just a partition image rather than an image of the full flash.
- The ESP32 will not permit any of the usual/easy methods of dumping out the rest of the flash partitions.
- The ESP32 has a single key burned into eFuses (the other slots are disabled; you can't add new keys!)
- The content of the flash is encrypted, transparently to the user, using that key and this cannot be disabled.
- Secure boot is enabled and the bootloader is immutable. If you change the bootloader, the hash will not match and the ROM will not jump to it.
- The bootloader (presumably) has a key baked into it that is used to verify the signature of the application firmware before jumping to it.

Or, put another way: OTA firmware files can be analyzed but are incomplete. Attempts to change the image files and flash them back to the device will - at best - not work. At worst, they will brick the device. This also goes for attempts to dump the full flash contents; Ubiquiti have locked this thing down pretty well.

Early revisions of the ESP32 [had various exploits](https://limitedresults.com/2019/11/pwn-the-esp32-forever-flash-encryption-and-sec-boot-keys-extraction/) but it's not clear if those still apply to the S3 variant used here.
Above, it looks like the chip in my unit was manufactured `222025` which is either mid-year 2022 or 2025 (probably) so it's not ancient and likely has the latest silicon fixes.
I would love to be proven wrong here!

### Did they cripple the device on purpose?

The author behind sfpw-tool reached out to me to share some additional details about the reverse engineering work that has been done on the firmware.
Specifically, comparing the code-flow between the old firmware vs new firmware w/r/t [passwords for locked SFP modules](https://github.com/vitaminmoo/sfpw-tool?tab=readme-ov-file#password-database).

> The newer flow just tries the specific entries that match the model that is inserted, and if those don't work it tries the single default. The old one tried the first match by model, then fell back to trying each unique one from the db.
>
> My tool has a command that auto-dumps the password database from firmware files directly, which is the other reason I'm looking for an old version.

I'm not sure what to make of this.
There are [many lists of SFP module passwords](https://forums.servethehome.com/index.php?threads/transceiver-password-collection.39458/) floating around the internet so it's not like this is a secret.

The SFPWizard wasn't even the first [device](https://eoinpk.blogspot.com/2014/05/raspberry-pi-and-programming-eeproms-on.html) on the [market](https://www.reveltronics.com/en/shop/80/12/chip-programmers/accessories-and-adapters/sfp-qsfp-xfp-adapter-for-programmer-80-detail) that could [program](https://ausoptic.com.au/blog/universal-transceivers-and-programmable-sfp-modules-flexbox-guide/) locked SFP modules so it's not like Ubiquiti were breaking new ground here.

I do not know if they got a nasty letter from some industry group / company and they changed how the device works in response or if it was a genuine mistake / they didn't intend to offer similar functionality to existing devices on the market.

**Regardless**, if you have a valid/signed 1.0.5 OTA firmware file, please keep it safe and get in touch with me.
That file can be used to downgrade the device and restore its "at launch" functionality.

