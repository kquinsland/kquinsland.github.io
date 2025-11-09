# Inside of the Pulse-Eight HDMI CEC Injector.

# Inside the Pulse-Eight HDMI CEC Injector

To make a long store short, Google recently pushed a few updates to my TV that made it slow as all hell and frustratingly unusable.
I finally got fed up with the laggy UI and dug out an old mini PC and installed [LibreELEC](https://libreelec.tv/) on it.

Everything worked perfectly _except_ for the lack of [CEC](https://en.wikipedia.org/wiki/Consumer_Electronics_Control) support on the mini PC.

This is a common enough problem that there's a few different devices out there that can add CEC support to a device that doesn't have it.
I chose the [Pulse-Eight HDMI CEC Injector](https://www.pulse-eight.com/p/104/usb-hdmi-cec-adapter) because it's natively supported by LibreELEC.

I was curious what was inside and I couldn't find any teardowns online so I decided to do one myself.
There's not much to the device and I'm not doing any serious reverse engineering here so let's call this a ["Two Minute Teardown"]({{<relref "/tags/two-minute-teardown/">}}).

{{< figure name="td01" >}}

{{< figure name="td02" >}}

I wasn't expecting much; I thought CEC was implemented via UART and not SPI so I was expecting to see a cheap FTDI knockoff chip or similar.
Turns out I was wrong and the device uses an [ATMEL 90USB162](https://www.microchip.com/en-us/product/at90usb162) which is a USB to SPI bridge.
I haven't looked too closely at the [firmware](https://github.com/Pulse-Eight/libcec) but I suspect there's some abstraction layer that makes it look like a [UART](#dmesg) to the OS.

{{< figure name="td03" >}}

{{< figure name="td04" >}}

{{< figure name="feat-td05" >}}

## `dmesg`

```shell
usb 5-4.1.2.3.2: new full-speed USB device number 2 using xhci_hcd
usb 5-4.1.2.3.2: New USB device found, idVendor=2548, idProduct=1002, bcdDevice=10.00
usb 5-4.1.2.3.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 5-4.1.2.3.2: Product: CEC Adapter
usb 5-4.1.2.3.2: Manufacturer: Pulse-Eight
usb 5-4.1.2.3.2: SerialNumber: v12
input: Pulse-Eight CEC Adapter as /devices/pci0000:00/0000:00:08.1/0000:0d:00.3/usb5/5-4/5-4.1/5-4.1.2/5-4.1.2.3/5-4.1.2.3.2/5-4.1.2.3.2:1.2/0003:2548:1002.0102/input/input195
hid-generic 0003:2548:1002.0102: input,hidraw10: USB HID v1.10 Mouse [Pulse-Eight CEC Adapter] on usb-0000:0d:00.3-4.1.2.3.2/input2
cdc_acm 5-4.1.2.3.2:1.0: ttyACM0: USB ACM device
usbcore: registered new interface driver cdc_acm
cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
```

