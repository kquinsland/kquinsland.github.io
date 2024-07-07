# Arizer XQ2 Teardown

<!-- markdownlint-disable-file MD002 -->
# Arizer XQ2 Teardown

{{<figure name="feat-product01">}}

A friend of mine reached out and asked me about automating some aspects of their [aroma therapy](https://www.webmd.com/balance/stress-management/aromatherapy-overview) treatment.

{{< admonition note >}}

I was not given permission to share the specifics of their medical condition or the larger treatment plan so this post is going to deal with _just_ the technical aspects of the device.

{{< /admonition >}}

The device they're using for aromatherapy is the [Arizer XQ2](https://arizer.com/xq2/) and we agreed that integration with their existing Home Assistant setup would be ideal.

I know nothing about aroma therapy but I do suffer from an obsession that compels me to integrate ESPHome with _everything_...I was intrigued!

At worst, I get a new toy to tear down and at best, I'd be able to deliver a solution that would make their life a little easier.

## The plan

First, the bad news.

There's next to no technical information about the XQ2 online.

The device has no radio connectivity so there's no FCC filings.
There are several reviews of the device online, but none of them tear it down or provide any technical information.

Now, the good news.

The device does have an IR remote control!

It is not difficult to capture the IR signals from the remote and replay them with an ESP32 or similar device.
This meant that the likely outcome would be an [open-loop control system](https://en.wikipedia.org/wiki/Open-loop_controller); building something to to send commands to the device would be trivial but confirming that a particular command had been executed would be tedious at best.

This put us in an awkward position.

After some back and forth, we agreed that closed-loop would be ideal, but open-loop control would be acceptable for their needs given that the device is meant to be used in close proximity to the user and already features a few safety features.

Unless there was some lucky break discovered while tearing down the device, I'd be stuck with emulating the IR remote control.
This isn't ideal, but it also means that there's virtually no risk that any of the built-in safety features would be bypassed/disabled.

## Teardown

The manufacturer has clearly designed this thing to be serviceable!

They [have a guide](https://vimeo.com/648841530) for replacing a common wear item (the ceramic heating element) and it turns out that the rest of the device is just as easy to access.

From the bottom of the device, there are 4 rubber feet that can be removed to reveal 4 screws.
Amazingly, these feet are NOT glued in _and_ they have tiny holes in them to allow the screws to be removed without removing the feet! 😚👌.

{{<figure name="td01">}}

Once the screws are removed, the bottom cover just lifts off.
Again, no glue, no clips, no nothing!

This reveals the main PCB. I've identified a few of the main ICs [below](#notable-chips-and-stuff).

{{<figure name="td02">}}

{{<figure name="td03">}}

Note that the DC jack is on a separate board that is easily removed.
This is a nice touch as it makes it easy to replace if it ever breaks; it's a standard 5.5mm OD barrel jack so sourcing a replacement should be easy.

{{<figure name="td04">}}

The PCB is _almost_ entirely single sided; only a few passives and connectors for the various peripherals are on the top side.

{{<figure name="td05">}}

The power supply isn't exotic so sourcing a replacement should be easy if it ever fails.
It's also worth noting that the 19V / 3.42A rated output is _really close_ to 20V / 3 A output profile from [USB-C PD 1.0 spec](https://web.archive.org/web/20190711152956/https://github.com/vi117/ppkos/blob/master/extdoc/usb_31_060115/USB%20Type-C/USB%20Type-C%20Specification%20Release%201.1.pdf) and totally within the adjustable range of a USB-C PD 2.0+ power supply.
It should be possible to adapt this device to use USB-C PD power with minimal effort if needed.

{{<figure name="td06">}}

All in all, I'm impressed with the build quality of the device!

It's well made and easy to service. No glue or clips to break, no hidden screws to find, no proprietary connectors to deal with.

The main PCB is well laid out, all components are marked with crisp silkscreen and nobody lasered off the markings on the ICs.

The most dangerous part of the device it the heating element and there are _multiple_ heat-shields between it and the user and all the heat-shields are easily replaceable!

The PCB designer also knew that MOSFETs can fail short so they use both a high _and_ low side FET to control the heating element!

Normally little piezo beepers are annoying but this one can be attenuated or disabled in software.

I was pleasantly surprised to find that the device _does_ have a UART and programming port onboard **but I was not given permission to probe further.**

My friend decided that IR control would be sufficient for their needs so I don't have any more information about the UART.

### Notable Chips and stuff

- `SWM260CBT7-50`: This is the main application processor. The manufacturer's [product details page](https://www.synwit.cn/gaishu313/) has both a (Chinese) [datasheet](https://www.synwit.cn/uploads/soft/20221222/1-221222144341C1.pdf) and a collection of [reference code](https://www.synwit.cn/uploads/soft/20231114/1-23111414411W47.rar).

Assuming the read/write protect fuses aren't blown, it should be possible to dump the firmware from this chip and possibly come up with an open-source replacement firmware for the device.
With the UART so nicely broken out, adding in an ESP32 or similar to provide wireless control should be trivial... :D.

- [`CMOS CMS6679`](https://item.szlcsc.com/5893434.html): MOSFET driver for the heating element.

- [`TPC8062`](https://www.mouser.com/ProductDetail/Toshiba/TPC8062-HLQCM?qs=QrGA9tEX17N%2FMHC76X%252B99w%3D%3D): MOSFET driver for the heating element.

## ESPHome IR Remote

I already mentioned this above but it's worth repeating!

{{< admonition warning "Open Loop controls suck" >}}
Basically, the code below emulates the IR remote control that comes with the device.
As there is no way for the device to communicate back to the original remote to say "I got your command" or "I'm busy right now, try again later", there's no way to know if the device is in the state you think it is.

The human operator can see the device's state and confirm that it's doing what they want... but this is _not_ the case for the ESPHome/Home Assistant integration.

The device also has a few safety features built in that the "remote" can't bypass.
If - somehow - the integration becomes fully broken, it's no different from sitting on the remote or otherwise spamming some signal.
In this case, the device will still operate safely.
{{< /admonition >}}

If you understand the risks and still want to proceed, here's a simple ESPHome configuration that will allow you to control the XQ2 via IR blaster.

```yaml
button:
  - name: "Power"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xE41B
          command_repeats: 1

  - name: "Fan Off"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xFE01
          command_repeats: 1

  - name: "Fan Low"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xF609
          command_repeats: 1

  - name: "Fan Med"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xFB04
          command_repeats: 1

  - name: "Fan High"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xF807
          command_repeats: 1

  - name: "Preset 1"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xEF10
          command_repeats: 1

  - name: "Preset 2"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xF906
          command_repeats: 1

  - name: "Preset 3"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xE01F
          command_repeats: 1

  - name: "Temp UP"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xFA05
          command_repeats: 1

  - name: "Temp DOWN"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xF50A
          command_repeats: 1

  - name: "Speaker"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xED12
          command_repeats: 1

  - name: "Auto Off Timer"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xF708
          command_repeats: 1

  - name: "Light"
    platform: template
    on_press:
      - remote_transmitter.transmit_nec:
          address: 0xFE01
          command: 0xE11E
          command_repeats: 1

remote_transmitter:
  id: transmitter
  pin:
    # Your values here will likely be different
    number: GPIO14
    mode:
      output: true
  carrier_duty_percent: 50%
```

