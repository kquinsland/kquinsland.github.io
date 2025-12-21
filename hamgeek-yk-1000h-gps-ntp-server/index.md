# #TwoMinuteTeardown: Hamgeek YK-1000H NTP

# Two Minute Teardown: Hamgeek YK-1000H NTP

{{<figure name="feat-render">}}

Curiosity got the better of me when browsing AliExpress and I spotted [this](https://www.aliexpress.us/item/3256808325833761.html?gatewayAdapt=glo2usa) GPS NTP server for under $100 USD.

I needed a cart-stuffer anyways so I ordered one as a backup for my [existing]({{<relref "posts/2022/10/yet-another-gps-pps-opnsense/index.md">}}) GPS/NTP server.

While waiting, I took a look around to see if there was any additional information about who makes this / if there was a custom firmware or similar.
Nope, there there is [_not_ a ton of information](https://www.dzombak.com/blog/2024/12/yk-1000h-ntp-server/) about the `YK-1000H` online.

Shortly after it arrived, I plugged it in and ... waited for it to get a GPS fix in an area where I know GPS works well.
After about 24 hours, still no fix!

That's when I noticed the warning that the seller had added to their product listing:

{{<figure name="warning-image">}}

I do not really understand why there's a $3 surcharge for the international version; I _assumed_ that all of the major GPS constellations use the same CDMA modulation in the same ~1.5GHz band so as long as you don't need the encrypted military signals, the radio hardware should be the same everywhere and just work everywhere.

Apparently not!
Oh well, the unofficial motto of AliExpress is _一分钱一分货_ ("Buyer Beware").

## Teardown

Start from the rear, remove the screws and the nut/grommet around the SMA connector.

{{<figure name="td-01">}}

You will not be able to slide the main PCB out because it's still connected to the front panel via a ribbon cable.

Then remove the front screws and then the main PCB can be slid out.

{{<figure name="td-02">}}

{{<figure name="td-03">}}

There's nothing of interest on the reverse side of the main PCB or the LCD board.

{{<figure name="td-04">}}

The cans on the GPS module are soldered on ... but not well.
With just a little bit of prying, they came right off... to reveal a very dusty interior.

{{<figure name="td-05">}}

I can't get an ID on the RF front-end chip, the [`MX-70G`](https://www.digikey.com/en/products/detail/macronix/MX29LV320EBTI-70G/2744812) next to the `UBX-G5000-BT` is almost certainly flash memory for the GPS module.
Almost none of the pads go anywhere; the two pads near that SMA connector are likely power and the other two pads that go anywhere (top left) are likely the serial TX/RX lines for configuration/debug.
It appears that the UART is broken out to that 3-pin header near the GPS module.

## ICs and Markings

- [`UBX-G5000-BT`](https://www.ixbt.com/monitor/images/aver-pilot/ubx-g5010.pdf): u-blox multi-constellation GPS receiver that absolutely should have worked anywhere in the world. Not sure why the seller insisted on the "international" version.
- [`WCH CH9126`](https://www.lcsc.com/datasheet/C717616.pdf): A dedicated SNTP client/server chip with built in ethernet MAC/PHY. Feels like a W5500 but instead of just TCP/UDP it has SNTP support built in.
- `74HC245KA`: A standard octal bus transceiver to help with data lines / level shifting.
- [`STC Micro STC15W4K56S4`](https://www.lcsc.com/datasheet/C91356.pdf): an 8051 based microcontroller to handle GPS parsing and general control of the device / driving the display.

The main PCB has some text and a QR code in the front/left corner:

```text
YK-1000P 卫星同步时钟主板
2024.11.25 Ver4.0
```

The QR code decodes to `YKDZ_250505_0255`

The "carrier" board with the CH9126 has:

```text
590679W_Y40_250925
```

The LCD/Display board has a QR code that decodes to `YKDZ_TFT_241220_0068`

Those are almost certainly production batch/date codes.
The pattern feels similar to what JLCPCB uses for their PCBs but that's just a guess / not uniquely identifying.

## Miscellaneous

I asked the seller for the software and they sent me a copy.
I do not know why they couldn't just have linked to it in the product listing ... but oh well.
Now you can download it w/o having to bother w/ the language barrier :).

- [Scanned Manual]({{< resource-ref "scanned-manual" >}}) (chinese and english)
- [OEM Manual (`YK-1000H卫星同步时钟产品说明书.pdf`)]({{< resource-ref "oem-manual" >}}) (chinese, only)
- [OEM Software (`90467 90479 YK-1000H 英文软件.exe`)]({{< resource-ref "oem-software" >}})

### Software

Partly because I didn't get the "international" version and partly because I don't have a proper Windows based RE environment set up, I wasn't able to dig into the software much.

Googling the very unique `CH9126_MODULE_V1.03` string turns up ... _one_ [reference](https://www.cnblogs.com/Lqqq123/p/18058274) post with some C code that probably matches what the OEM software is doing to configure the module.

I have archived the reference post [here](https://web.archive.org/save/https://www.cnblogs.com/Lqqq123/p/18058274) and [here]({{< resource-ref "random-reference-code" >}}).
That reference post has a link to a [`CH9126ConfigTool.zip`]({{< resource-ref "random-reference-sdk" >}}) which appears to be a reference implementation/SDK with a compiled `.exe` as well.
The `.exe` isn't the same one as what the seller sent me ... but the icon is identical so they're probably from the same source.

The `strings` output did raise an eyebrow:

- The common UART baud rate strings is interesting, probably for configuring the comms to the main microcontroller.
- No obvious certificates or cryptographic strings; firmware might not be signed?
- There is some mechanism for saving configuration. Presumably that's a cheap way to load arbitrary bytes into the device... :)

```shell
❯ strings 90467\ 90479\ YK-1000H\ 英文软件.exe
SNTP CHANNEL
SNTP
Space
Mark
Even
None
Operation Status:
(<=1024)
Data Format:
Rx Pack TimeOut:
Rx Pack Len:
Parity:
Stop Bit:
Data Bit:
Baud:
PORT CONFIG
GateWay:
Mask:
Name:
NET CONFIG
SAVE CONFIG
LOAD CONFIG
CONFIG
RESET
SEARCH
Module List(Double Click to get configuration)
REFRESH
Adapter:
Name
Operation Status:
%d.%s
bind error : %d
ip:%s  mac: %02X:%02X:%02X:%02X:%02X:%02X
socket error : %d
Config Successful!
Reset Successful!
Successfully Get Configuration On Line %d
Search Device Successful!
CH9126_MODULE_V1.03
recvfrom error : %d
Receive Failed...
sendto error : %d
<...>
Searching Devices...
Create Socket Failed, Please Retry...
Please Choose One Module From Module List,If There Is None,Please Search First
No Valid Rows Selected...
Getting Device Configuration...
The Length Of The Module Name Is Illegal,Please Retry...
Rx Pack Len is between 0 and 1024!Please Re-enter...
1024
Configuring...
Save Configuration Successful!
%s.cfg
saved
CFG Files (*.cfg)|*.cfg||
Load Successful!
saved.cfg
CFG Files (*.cfg)|*.cfg|
open
.\Help.txt
Pulse Output:
Polling Interval:
Dest IP:
Mode:
Random
Local Port:
Dest Prot:
<...>
1200
2400
4800
9600
14400
19200
38400
57600
115200
230400
460800
921600
Mark
Space
ASCII
TCP_Sever
TCP_Client
UDP_Server
UDP_Client
KeepSocket
DiscardSocket
StoreAllData
StoreLastData
StoreNoData
SNTP_Server
SNTP_Client
```

