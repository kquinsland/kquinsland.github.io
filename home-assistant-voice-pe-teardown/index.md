# #TwoMinuteTeardown: Home Assistant Voice - Preview Edition


# Home Assistant Voice: Preview Edition teardown

This is a quick [#TwoMinuteTeardown]({{< relref "tags/two-minute-teardown" >}}) post for the recently released [Home Assistant Voice Node - Preview edition](https://www.home-assistant.io/voice-pe/).

Normally, I'd be doing a full teardown, but this is a little different; the Home Assistant team has done a _lot_ of the work out in the open.
With just a quick Google, you can find official:

- teardown instructions [here](https://voice-pe.home-assistant.io/guides/disassemble/).
- source code [here](https://github.com/esphome/home-assistant-voice-pe).
- 3D printable case [files](https://www.printables.com/model/1110526-home-assistant-voice-preview-edition-enclosure).
- Instructions for adding custom devices [here](https://voice-pe.home-assistant.io/guides/grove_port/).

I am grateful for such a well-documented and open project.
Truly, by hackers for hackers!

## Teardown

See the official instructions if you can't figure it out.

There's not much to the device, really.
The top half hosts the physical buttons and the microphones.
The thin, metal-looking piece in the top right is the antenna.

In the bottom right, there's a small switch that appears to mux between the main application processor (ESP32) and the audio processor.

I wonder weather this is "leftover" from development and they haven't gotten around to simplifying the BOM yet or if this is intentionally populated for users to re-program the audio processor.

{{<figure name="pcb_top">}}

All of the "fun" stuff is on the bottom half.

Immediately, you notice the PCB has multiple _populated_ pin headers for hacking. Bonus, they're all labeled!

The layout and BOM is about what you'd expect for a ~$50 single-purpose device.

Where possible, I've [identified](#ics) the big ICs.

{{<figure name="feat_pcb_btm">}}

### wishful thinking

In a word: Ethernet.

Most of the ESP32 product line does not have support for ethernet via [RMII](https://en.wikipedia.org/wiki/Media-independent_interface#RMII) (unfortunately) but there is always SPI based ethernet.

There are a few areas where the PCB is bare/empty and with a few tweaks to the layout, it would have been possible to also expose SPI headers next to the [grove connector](https://wiki.seeedstudio.com/Grove-I2C_Hub/).

Assuming there's a spare SPI peripheral to use, SPI based ethernet would have latency and throughput as good as or better than WiFi.

It would have been a nice option for those of us who prefer the reliability of wired connections.

Oh well.

## ICs

This is just the big ones that were going to be easy-ish to positively identify.
I skipped a few of the super small ICs that had a few characters (at most!) of identification on them.

Two flash chips, one for the ESP32 and one for the "ai" chip which is maybe doing detection and is almost certainly doing some other DSP stuff.
TI parts for the audio codec and a few miscellaneous bits as well.

### zbit semiconductor ZB25VQ128

```text
AN2414
25VQ128DSJG
P3S952
```

[This](https://linux-chenxing.org/pioneer3/sv50pd/ZB25VQ128.pdf) is a clone of a super common winbond nor flash chip with the same part number.

### zbit semiconductor ZB25VQ32

```text
KN2444
25VQ32DSJG
EQ4537
```

[Same](https://semic-boutique.com/wp-content/uploads/2016/05/ZB25VQ32.pdf) as above, but smaller.

### TI TLV320AIC3204

[This](https://www.ti.com/product/TLV320AIC3204) is the audio codec.

```text
AIC
3204
TI 48I
A69Y G4
```

### TI SN74LVC125A

[This](https://www.ti.com/product/SN74LVC125A) is a pretty common buffer IC and I don't know for sure what it's being used for.
I am 99% certain that the LEDs under the rotary encoder are neopixels and those can be driven directly from the ESP32.
The only other thing that comes to mind is a "switch" to physically cut power to the microphones when the user has engaged the mute switch.

```text
LC125A
TI44K
C237
9F
```

### XMOS XU316

The [XU316-1024-QF60A](https://www.xmos.com/download/XU316-1024-QF60A-xcore_ai-Datasheet%2826%29.pdf) is an interesting chip.
The product name and datasheet suggest that it's instrumental in "ai" workloads 🙄 but the datasheet is a bit more concrete:

> The xcore.ai series is a comprehensive range of 32-bit multicore microcontrollers that brings the low latency and timing determinism of the xCORE architecture to mainstream embedded applications.
> Unlike conventional microcontrollers, xCORE multicore microcontrollers execute multiple real-time tasks simultaneously and communicate between tasks using a high speed network.
> Because xCORE multicore microcontrollers are completely deterministic when executing from internal memory, you can write software to implement functions that traditionally require dedicated hardware.

This is doing real time audio processing - presumably to cancel background noise/echo - and might also be doing wake word detection.

```text
XMOS
V16A0
GT2446P2
TMCY48.00
```

### Espressif ESP32-S3

This chip needs no introduction!

```text
ESP32-S3
242024
R8MRK668000
UC00MBE423
```

### The little guys

All the ICs that are too small to have full/meaningful part numbers.

```text
FE
NZ
```

```text
f9
GkA
```

```text
CEDTI
```

Given the proximity to both the speaker connector and the [audio codec](#ti-tlv320aic3204), I am guessing this is an audio amplifier, probably also from TI.

```text
TI 45
AYK
```

