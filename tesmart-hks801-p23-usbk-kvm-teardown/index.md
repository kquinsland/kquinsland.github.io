# TESmart HKS801-P23-USBK KVM Teardown

# TESmart HKS801-P23-USBK KVM Teardown

Recently, one of the USB ports on [my previous KVM switch]({{< relref "posts/2022/04/hdmi-kvm-teardown-and-esphome" >}}) failed. Since it's a critical piece of equipment, I needed a replacement the same day. The original AliExpress product page was no longer available; the seller seems to have closed up shop entirely.

Fortunately, the OEM—`XUFUNG`—has released an updated version, now sold in the US as the [TESmart HKS801-P23-USBK](https://www.tesmart.com/products/hks801-p23). It looks visually identical to the previous KVM switch, but the USB ports now feature the characteristic blue color indicating USB 3.0 support.

{{<figure name="oem01">}}

{{<figure name="oem02">}}

{{<figure name="oem03">}}

{{< admonition note >}}

Despite every PCB featuring a clear `XUFUNG` label, that specific string doesn't appear in many places online! A quick Google search only returns my prior post and [this teardown of another TESmart KVM](https://dancharblog.wordpress.com/2025/05/02/tesmart-hdc202-x24-thunderbolt-4-kvm-teardown-and-review/). You should check out the internals of that one too! Because it uses Thunderbolt, the internals are quite a bit more complicated.
{{< /admonition >}}

## What's inside?

{{< admonition tip "TL;DR">}}
It's the same architecture as before, but with some minor component changes and corresponding PCB layout updates.
{{< /admonition >}}

As before, two main PCBs are stacked: the top houses the main microcontroller and HDMI ports, while the bottom contains the USB ports and power supply.

{{<figure name="td01">}}

{{<figure name="td02">}}
{{<figure name="td03">}}
{{<figure name="td04">}}

It's interesting to see that the power rails differ: 5V for the 4K version, and apparently there's an 8K version that uses 12V. It's also nice that the RX/TX lines are explicitly labeled; it makes things a bit easier to hack on.

{{<figure name="td05">}}

## Chips

In no particular order, here are the main ICs I identified on the PCB:

- [RTS5411s](https://lcsc.com/product-detail/USB-Converters_Realtek-Semicon-RTS5411S-GR_C5356838.html) - USB 3.1 Super-Speed HUB Controller
cx32l003f8

- F68B / A1V8 - Not sure, can't find any positive ID. There's one for every RTS5411S port, though, and some traces do daisy chain between the individual F68B chips so i'm guessing they're related to USB and possibly a buffer/signal conditioner?
  - Interestingly enough, I don't see any traces from this chip to the "slow" USB ports that are meant to handle the keyboard/mouse. Those are monitored by the CH545L which is probably looking for the keyboard/mouse signals to trigger switching between banks.

- [WCH CH545L](https://lcsc.com/product-detail/Microcontrollers-MCU-MPU-SOC_WCH-Jiangsu-Qin-Heng-CH545L_C19183926.html?s_z=n_C19183926) - USB multi-host multi-device enhanced E8051 MCU

- [CX32L003F8](https://www.zbitsemi.com/upload/file/20200903/20200903105452_99571.pdf) - ARM® Cortex®-M0+ 32-bit
  - Used in a few different locations: front panel to scan buttons, update the display, monitor IR signals.

- [WCH CH578M](https://www.wch-ic.com/products/CH578_7.html) - ARM® Cortex®-M0 Core Bluetooth Low Energy MCU
  - This is the micro on the LAN port; makes sense as it has a built in 10M Ethernet PHY
  - Assuming there's no restriction that prevents the ethernet PHY and BT PHY from being used at the same time, this KVM could have ditched the IR remote and used a phone app instead...
  - Previously, this was handled with a [ST 8S003f3p6](https://www.st.com/en/microcontrollers-microprocessors/stm8s003f3.html) and a [WCH CH395Q](https://jlcpcb.com/partdetail/WCH_Jiangsu_Qin_Heng-CH395Q/C87390) so the LAN controller has definitely had a BOM consolidation!

- [Chipsea CS32F031C8T6](https://en.chipsea.com/product/details/?choice_id=1079) - ARM® Cortex®-M0 MCU
  - Happens to be the exact same micro as the previous KVM and seems to be featured in Dan's KVM, too

- [WS3232ECN](https://jlcpcb.com/partdetail/GEC-WS3232ECN/C2973151) - Generic RS232 transceiver
  - Previously this was done with a `SIPEX SP3223EEX` which is not the same footprint as the WS3232ECN.. so that's a minor layout change

- [IT66341TE](https://web.archive.org/web/20240523025020/https://www.ite.com.tw/en/product/view?mid=99) - 4 IN to 1 OUT HDMI2.0 18Gb/s Switch.
  - Still can't find any formal technical info on this chip, probably due to [HDCP](https://en.wikipedia.org/wiki/High-bandwidth_Digital_Content_Protection) restrictions.
  - Same as the previous KVM, but that one also used an IT66321E, which is not present on this PCB.

