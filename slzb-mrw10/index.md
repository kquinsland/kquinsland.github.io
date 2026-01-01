# #TwoMinuteTeardown: SLZB-MRW10

# Two Minute Teardown: SMLIGHT SLZB-MRW10

To make a very long story short, I need to move away from my _very_ old [802.15.4](https://en.wikipedia.org/wiki/IEEE_802.15.4) radios plugged into a _very_ old Raspberry Pi for my smart home setup.
After some research, the [SMLIGHT SLZB-MRW10](https://smlight.tech/product/slzbmrw10) is new, uses modern radios and ships with firmware that [makes it trivial to move packet processing into compute distributed on the other end of the network](https://www.zigbee2mqtt.io/advanced/remote-adapter/connect_to_a_remote_adapter.html).

Specifically, the [Silicon Labs EFR32ZG23](https://www.silabs.com/wireless/z-wave/800-series-modem-soc) implements Z-Wave 800 (+ Long Range) and the [Texas Instruments CC2674P10](https://www.ti.com/product/CC2674P10) implements Zigbee 3.0 or Thread.

Shame you can't do both Zigbee and Thread at the same time, but I'm not in any [hurry](https://news.ycombinator.com/item?id=38763743) to switch to [Matter](https://en.wikipedia.org/wiki/Matter_(standard)) for a [long](https://news.ycombinator.com/item?id=44508740) [list](https://news.ycombinator.com/item?id=38763743) of [reasons](https://news.ycombinator.com/item?id=44508727) so I'll happily stick with the Zigbee firmware.

The core is an ESP32 that handles the network connectivity and BT/WiFi radios.

{{<figure name="render">}}

In the box you get the main radio unit, antennas and some misc mounting hardware / a template to help with spacing for the wall mount.
I love it when they include a template instead of just saying "measure and drill holes yourself". Bonus; the QR code links to the documentation page.

{{<figure name="td01">}}

No screws so the back just snaps right off.

{{<figure name="td02">}}

{{<figure name="td03">}}

{{<figure name="td04">}}

SMLIGHT does have other devices that are similar form factor so I guess it makes sense to re-use the PCB and outfit it with a programming resistor.
I don't think this is for indicating the radio regions because I can change that in software.

{{<figure name="td05">}}

{{<figure name="td06">}}

{{<figure name="td07">}}

{{<figure name="td08">}}

{{<figure name="td09">}}

## Misc

Aside from the main ICs/Radios mentioned above, there are a few other interesting components:

- [`Texas Instruments MUX1574`](https://www.ti.com/product/TMUX1574): not sure what this is doing here... unless it's managing the 6 LEDs scattered around the board? There appear to be two LEDs for each of: the ESP32, the CC2674P10, and the EFR32ZG23 which makes for a total of 6 LEDs ... which is two more than the MUX1574 can handle.
- [`SMSC 8720A`](https://ww1.microchip.com/downloads/en/devicedoc/8720a.pdf): Ethernet PHY, same as the ones used in the [LED Controllers I recently tore down]({{< relref "/posts/2025/12/led-controllers" >}}).
- [`Toll Semiconductor TMI7321`](https://www.toll-semi.com/PDPoEICir2/214.html): PoE controller IC, also used in one of the LED controllers.

### `dmesg`

The USB-C port is not just for power; as far as I can tell, it's wired to a [CP210](https://www.silabs.com/documents/public/data-sheets/CP2102-9.pdf) which is connected to the ESP32.
Firmware on the ESP32 then proxies UART traffic from the two radios allowing you to interact with all radios over a single serial connection.

Interesting to see that they bothered to set the product, manufacturer, and serial number strings on the USB device instead of leaving them w/ the default values.

```shell
[176519.261646] usb 5-4.1.4: new full-speed USB device number 74 using xhci_hcd
[176519.370686] usb 5-4.1.4: New USB device found, idVendor=10c4, idProduct=ea60, bcdDevice= 1.00
[176519.370691] usb 5-4.1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[176519.370693] usb 5-4.1.4: Product: SMLIGHT SLZB-MRW10
[176519.370695] usb 5-4.1.4: Manufacturer: SMLIGHT
[176519.370697] usb 5-4.1.4: SerialNumber: 78<....>10
[176519.393095] usbcore: registered new interface driver cp210x
[176519.393110] usbserial: USB Serial support registered for cp210x
[176519.393159] cp210x 5-4.1.4:1.0: cp210x converter detected
[176519.400860] usb 5-4.1.4: cp210x converter now attached to ttyUSB0
```

