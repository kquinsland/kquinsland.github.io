# #TwoMinuteTeardown: Panel Mount Liquid Level Indicator

# Generic liquid level meter teardown

I ordered this part years ago for a project I eventually canceled.

I can't use it for anything, and it's just taking up space, so I'm going to open it up before I recycle it.

The Ali Express listing is a 404 so I don't have a link to share or really any information about it.

The main thing to take away is that it's meant to interface with a liquid level sensor.
It will display the level of liquid in a tank and - optionally - trigger a few relays based on the level.

{{<figure name="feat_front">}}

This is a generic case; not all screw terminals pictured here are actually wired up to anything on this model.

{{<figure name="rear">}}

{{<figure name="side">}}

The main body slides off pretty easily.
There are two PCBs inside: one for the display and one to handle the relays and sensor input.

{{<figure name="td01">}}

The display PCB is pretty simple; no active electronics and a tonne of LEDs.
The bar graph LEDs are almost certainly [charlieplexed](https://en.wikipedia.org/wiki/Charlieplexing) but the 7 segment might have an integrated driver.

{{<figure name="td02">}}

The screw terminals on the rear of the case are directly soldered to the main PCB so I'm not going to be able to take it apart any further without disordering things... defeating the spirit of the [#TwoMinuteTeardown]({{< relref "tags/two-minute-teardown" >}}).

{{<figure name="td03">}}

Looking at the top left, you can see the few pins for the 7-segment displays; a total of 12 pins.
Of those 12, some don't have traces leading to them so they're probably just for mechanical support. There are far too few remaining pins for even basic charlieplexing so I'm guessing there's an integrated driver of some kind. Probably a [max7219](https://www.analog.com/media/en/technical-documentation/data-sheets/max7219-max7221.pdf) or similar

## The main ICs

Really just the one main application processor and a few support ICs.

- [`MA803AS2`](https://www.jotrin.com/product/parts/MA803AS2) - 8051 based microcontroller that's running the show
- [`HEF4051B`](https://www.alldatasheet.com/datasheet-pdf/pdf/1679343/NEXPERIA/HEF4051B.html) - 8-channel analog multiplexer, probably for the display
- `74hc164` - 8-bit shift register, probably for the display

