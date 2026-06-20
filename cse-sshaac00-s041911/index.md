# #TwoMinuteTeardown: Corporate Systems Engineering SSHAAC00-S041911

Utility grade Air Conditioning Load Shed Device teardown.

Canonical: https://karlquinsland.com/cse-sshaac00-s041911/
Published: 2026-01-24
Updated: 2026-01-24
Tags: two-minute-teardown


---

# Corporate Systems Engineering SSHAAC00-S041911

Electric utilities can employ ["load shed" devices](http://corporatesystems.com/?page_id=64) throughout their grid to manage load.
These devices are usually found on air conditioners, water heaters, pool pumps and other large electrical loads.
Typically they'll offer you a small financial incentive to allow the utility to cycle your device off for short periods of time during peak demand events.

The Corporate Systems Engineering SSHAAC00-S041911 is one such device.

As you might expect from a single purpose device, the internals are pretty straightforward.
There is a modem/radio side to communicate with the utility, and a power switching side to understand and control the load.

This unit fell into my hands after it was taken off an air conditioner during a service call.

![_td01](https://karlquinsland.com/cse-sshaac00-s041911/images/td01.webp)


A bit counter intuitively, the entire assembly is attached to the lid of the enclosure and not the main body.
This will start to make more sense when looking at a few other details below.

![_td02](https://karlquinsland.com/cse-sshaac00-s041911/images/td02.webp)


The purple and grey wires ran off to a CT clamp that was installed around one of the AC compressor lines.

![_td03](https://karlquinsland.com/cse-sshaac00-s041911/images/td03.webp)


Small patch style antenna (`WPANT10170-S2B`) for the radio attached to the  side of the 'lid' with a sticky pad.

![_td04](https://karlquinsland.com/cse-sshaac00-s041911/images/td04.webp)


Taking a closer look at the main PCB and going in no particular order.

![_power01](https://karlquinsland.com/cse-sshaac00-s041911/images/power01.webp)


![_power02](https://karlquinsland.com/cse-sshaac00-s041911/images/power02.webp)


The PCB has cutouts for a few different LEDs, not all are populated in this unit.

![LEDs face 'down' to shine through the cutouts in the PCB and the lid.](https://karlquinsland.com/cse-sshaac00-s041911/images/power03.webp)

_LEDs face 'down' to shine through the cutouts in the PCB and the lid._


![Layer indicator on the right is a nice touch.](https://karlquinsland.com/cse-sshaac00-s041911/images/power04.webp)

_Layer indicator on the right is a nice touch._


There really isn't much to see on the bottom of the main PCB.
**Big** traces for the mains side of things, lots of clearance around everything else and a lot of ground plane stapling/via-stitching on the low voltage side of things.

![_power05](https://karlquinsland.com/cse-sshaac00-s041911/images/power05.webp)


Both the transformer and the single relay are `Zettler` parts.
The caps are `Nichicon`.
No expense spared it seems!

![_power06](https://karlquinsland.com/cse-sshaac00-s041911/images/power06.webp)


![_power07](https://karlquinsland.com/cse-sshaac00-s041911/images/power07.webp)


![_power08](https://karlquinsland.com/cse-sshaac00-s041911/images/power08.webp)


The ARM seems to do house keeping... although it's probably overkill for what this device needs to do.
The 'tall' pin headers are where the modem module mates.

![_power09](https://karlquinsland.com/cse-sshaac00-s041911/images/power09.webp)


Speaking of the modem module...

![The 2x8 pin header (center, bottom) mates to the main PCB.](https://karlquinsland.com/cse-sshaac00-s041911/images/modem01.webp)

_The 2x8 pin header (center, bottom) mates to the main PCB._


![_modem02](https://karlquinsland.com/cse-sshaac00-s041911/images/modem02.webp)


![_modem03](https://karlquinsland.com/cse-sshaac00-s041911/images/modem03.webp)


![_modem04](https://karlquinsland.com/cse-sshaac00-s041911/images/modem04.webp)


## ICs

In - roughly - the order they appear in the photos above.

### Power

```text
324
MZJF150
```

The [LM324](https://www.st.com/en/amplifiers-and-comparators/lm324.html) quad op-amp. For what, exactly, I'm not sure.

```text
ATMEL
M90E26-YU
2249
2249W66
```

The [M90E26](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-46002-SE-M90E26-Datasheet.pdf) is a _very_ accurate single phase energy metering IC.

```text
Zettler
AZ943-1CH
```

[AZ943-1CH](https://www.azettler.com/pdfs/az943.pdf) is the 'dry contract' wired in series with the thermostat to switch the AC compressor on/off.

```text
L2207
LTV827
```

[Lite-On LTV-827](https://datasheet.octopart.com/LTV-827-Lite-On-datasheet-7282434.pdf) optoisolator. This is almost certainly for isolating the `M90E26` from the rest of the low voltage side of things.

```text
MIC
69153YME
```

[Microchip MIC69153](https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/DataSheets/MIC69153-Single-Supply-VIN-LOW-VIN-LOW-VOUT-1.5A-LDO-DS20006019A.pdf) is power conditioning for the housekeeping MCU.

```text
0AD8
L22676
```

[Texas Instruments LM22676](https://www.ti.com/lit/gpn/LM22676-Q1) is power supply for the low voltage side of things.

```text
32F100
RDT6B
GQ277 93
```

[ST STM32F](https://www.st.com/en/microcontrollers-microprocessors/stm32f100-value-line.html) is the main housekeeping MCU.

```text
95M01RP
K109K
```

[ST M95M01](https://www.st.com/en/memories/m95m01-r.html) is flash memory for the MCU.

```text
PZGM
TI 268
```

[Texas Instruments TPS7A8300](https://www.ti.com/lit/ds/symlink/tps7a8300.pdf) is another power conditioning IC.

### Modem

The modem module appears to be `Itron` [OWS-NIC511-06](https://fccid.io/OWS-NIC511-06).
As far as I can tell, `Itron` acquired `Silver Spring Networks` who originally developed the module.
The PCB has quite a few unpopulated footprints and design features; each customer/region(?) gets a slightly different configuration based on their needs.
At least in _this_ configuration, it uses 915MHz ISM band and a mesh technology that Itron calls [`OpenWay`](http://corporatesystems.com/?page_id=47).

I'll defer you to the excellent work of [@RECESSIM](https://www.youtube.com/@RECESSIM) for some details on smart meters and mesh networks.

```text
Silver Spring Networks
130-0117-01
Rev 8.0
VQ239257
GW00E 07 E
```

Appears to be ["endpoint security module"](https://csrc.nist.gov/CSRC/media/projects/cryptographic-module-validation-program/documents/security-policies/140sp3445.pdf).

```text
ESMT
M14D128168A
AZY2 -2.5 BI
P2A2509 216
```

[DDR 2 memory](https://www.esmt.com.tw/upload/pdf/ESMT/datasheets/M14D128168A(2Y)_operation%20temperature%20condition%20-40~95%C2%B0C.pdf)

```text
RFFM
SSN1
VZ7K
```

I can't find anything definitive on this one but it's clearly a front end module for the RF side of things.

```text
Atmel
86RF215
```

Atmel [AT86RF215](https://www.microchip.com/en-us/product/at86rf215) Sub-GHz / 2.4GHz Transceiver and I/Q Radio

```text
SiGe
2435L
24OCM
```

[SkyWorks SE2435L](https://www.skyworksinc.com/products/front-end-modules/se2435l) is a power amplifier for the 902-928MHz ISM band.

