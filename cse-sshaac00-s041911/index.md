# #TwoMinuteTeardown: Corporate Systems Engineering SSHAAC00-S041911

# Corporate Systems Engineering SSHAAC00-S041911

Electric utilities can employ ["load shed" devices](http://corporatesystems.com/?page_id=64) throughout their grid to manage load.
These devices are usually found on air conditioners, water heaters, pool pumps and other large electrical loads.
Typically they'll offer you a small financial incentive to allow the utility to cycle your device off for short periods of time during peak demand events.

The Corporate Systems Engineering SSHAAC00-S041911 is one such device.

As you might expect from a single purpose device, the internals are pretty straightforward.
There is a modem/radio side to communicate with the utility, and a power switching side to understand and control the load.

This unit fell into my hands after it was taken off an air conditioner during a service call.

{{<figure name="_td01">}}

A bit counter intuitively, the entire assembly is attached to the lid of the enclosure and not the main body.
This will start to make more sense when looking at a few other details below.

{{<figure name="_td02">}}

The purple and grey wires ran off to a CT clamp that was installed around one of the AC compressor lines.

{{<figure name="_td03">}}

Small patch style antenna (`WPANT10170-S2B`) for the radio attached to the  side of the 'lid' with a sticky pad.

{{<figure name="_td04">}}

Taking a closer look at the main PCB and going in no particular order.

{{<figure name="_power01">}}

{{<figure name="_power02">}}

The PCB has cutouts for a few different LEDs, not all are populated in this unit.

{{<figure name="_power03">}}

{{<figure name="_power04">}}

There really isn't much to see on the bottom of the main PCB.
**Big** traces for the mains side of things, lots of clearance around everything else and a lot of ground plane stapling/via-stitching on the low voltage side of things.

{{<figure name="_power05">}}

Both the transformer and the single relay are `Zettler` parts.
The caps are `Nichicon`.
No expense spared it seems!

{{<figure name="_power06">}}

{{<figure name="_power07">}}

{{<figure name="_power08">}}

The ARM seems to do house keeping... although it's probably overkill for what this device needs to do.
The 'tall' pin headers are where the modem module mates.

{{<figure name="_power09">}}

Speaking of the modem module...

{{<figure name="_modem01">}}

{{<figure name="_modem02">}}

{{<figure name="_modem03">}}

{{<figure name="_modem04">}}

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

