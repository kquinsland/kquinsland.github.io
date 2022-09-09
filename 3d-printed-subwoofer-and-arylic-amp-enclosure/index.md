# I made a thing: Yet another 3d printed speaker

<!-- markdownlint-disable-file MD002 -->

# Background

Long story made **_very_** short: the amplifier inside my ancient 2.1 desktop speakers died and I couldn't find anything "off the shelf" that would serve as a suitable replacement _and_ integrate well with Home Assistant.

So if you can't buy it, you have to build it!
And as it turns out, there's a whole community of audiophiles that have published designs on all the usual places you'd find free designs for makers.
Similarly, there's a few companies that seems to specialize in audio electronics aimed _specifically at_ people that are building their own speaker systems and just want someone else to handle the electronics and software.

This post will cover the design tweaks I made to an existing subwoofer assembly and an original amplifier electronics/enclosure design.

## The components

There are two assemblies in this build: the subwoofer and the receiver/power-electronics enclosure.

The subwoofer is a remix of the amazing [`Skeleton Waveguide Subwoofer Speaker`](https://www.printables.com/model/84521-skeleton-waveguide-subwoofer-speaker-fusion-360-ar) by [`zx82net`](https://www.printables.com/social/107245-zx82net/about) which is a remix of [`"HexiBox" Tang Band W3-1876S Subwoofer Enclosure`](https://www.printables.com/model/190762-hexibox-series-tang-band-w3-1876s-microsub)
by [`hexibase`](https://www.printables.com/social/264072-hexibase/about).

As for the drive electronics, I elected to use an [Up2Stream Amp 2.1 - Multiroom Wireless 2.1 Amplifier Board](https://www.arylic.com/products/up2stream-amp-2-1-amp-board).
This appears to be based on a MediaTek based module module from [`LinkPlay`](https://www.linkplay.com/).

I could do a whole _series_ of posts based on the firmware alone but for now I'll just say that it is 1) trivial to root and 2) **should not be exposed directly to the internet for any reason whatsoever.**

I'm not equipped to verify any of the claims made by the original subwoofer designer so I can only offer my subjective take; this subwoofer offers more wubz at greater distances and that's _before_ I add +10db to the bass via the equalizer on the amp!

## Tweaks

{{<figure name="cad_sub_gaskets" >}}

### Wiring

- There is now an integrated channel for standard 16 AWG speaker wire that runs from the main cavity, through each baffle, to the exterior.
  - This cable channel also increases rigidity which aids in printing and assembly.
- The rear of the subwoofer body features some attachment points for a simple enclosure for speaker hookups.
  - I am using RCA jacks but the enclosure could be easily modified to work with any similar connector.
  - The enclosure mounts directly over the cable channel so all wires are protected/hidden.

### Assembly

You no longer need to use goopy adhesives to seal the panels to the body! The sub body and panels have been modified to accommodate a thin 3d printed sheet of TPU or similar material to act as a gasket.

There is also a subtle channel cut into the sub body which is meant for gasket cord as a fall back.

{{<figure name="cad_gasket-cord_overview" >}}
{{<figure name="cad_gasket-cord_closeup" >}}

### Mounting

- The sides of the subwoofer feature a 100x100mm [VESA/FDMI](https://en.wikipedia.org/wiki/Flat_Display_Mounting_Interface) mounting pattern.
  - When printing with the holes facing the print bed, support material all but entirely obscures their existence.

{{<figure name="sub_feat_panel_vesa">}}

# Build

{{< admonition info >}}
This is an abridged assembly guide and Bill of Materials. I have included only what's different/new/specific to this build. Consult the various build guides and BOMs for the original subwoofer so you understand what _else_ is required.
{{< /admonition >}}

## BOM

### Subwoofer

Very little about the subwoofer BOM has changed from the original design.

- **Optional**: [(1 mm) Buna-N O-Ring Cord Stock, 70A Durometer](https://www.amazon.com/gp/product/B00QVB6QPU/)
  - You will not need this if you're printing TPU gaskets or using some glue / caulk.
  - You will want some superglue or similar to help secure the gasket cord as you insert it into the channels.

- 16-Gauge Speaker Wire Cable, ~18 inches/40 cm.
  - The cable channel was specifically designed for 16 AWG speaker wire with the dimensions 6mm x 3mm. Any wire can work but if there's a gap between the wire jacket and the channel walls, you'll need to fill the gap / seal the channel.
  - 18 inches will allow for some relative comfort during assembly. You can get away with less but it will make some things a bit harder than needed.

- 28+ `M4x 12mm` screws.
  - The original design didn't include any specific screws so I explicitly chose [McMaster # 90666A115](https://www.mcmaster.com/90666A115/).
  - You will need at least 6 screws to secure the speaker to the sub and 11 to properly secure each panel to the subwoofer body.
  - If you choose to install the electronics enclosure at the rear of the sub, an additional 4 screws will be needed.

- RCA jack: there many different types out there but the parts are meant to be used with [these jacks](https://www.aliexpress.com/item/2255799906959194.html).

### Amplifier enclosure

- The heart of the build is the [Up2stream Amp 2.1](https://www.aliexpress.com/item/2255801111428335.html) module.
  - The AliExpress listing has a few versions. I was not planning on using the IR remote and I didn't want an external PSU so I chose to order _just_ the board.

Power Supply:

- The amplifier enclosure has room for a 24V 4.5A MeanWell [LRS-100](https://www.aliexpress.com/item/3256803302407060.html)
- To connect the DC output of the LRS-100, you'll need a [female 3.96MM 2 pin](https://www.amazon.com/gp/product/B0B37H6R56/) connector to mate with the DC in on the amplifier.

Panel mount connectors:

- RCA jacks: I used the same part number from above, just ordered a few different colors for the R/L/S channels.

- Powercon True. Rather than a traditional IEC style or simple DC barrel jack + external PSU, I opted for a [Skookum](https://en.wikipedia.org/wiki/Skookum) power connector. Like with the RCA jacks, there are many different parts from several manufacturers that will probably work but I specifically used [this set from Ali Express](https://www.aliexpress.com/item/2251832762646533.html).

- RJ45: There are likely several that will work, but I explicitly used [this part from Ali Express](https://www.aliexpress.com/item/2255799916665398.html).

- USB2.0: There are likely several that will work, but I explicitly used [this part from Ali Express](https://www.aliexpress.com/item/2255799916028028.html).

- USB extension cable: You can find all sorts of adapters in every conceivable orientation on Ali Express from stores like [this](https://www.aliexpress.com/item/2255801102773472.html). For this particular project, you will need a 15 cm flat flex cable (`FFC`) and 2x of the "straight" USB male connectors.

Screws:

- The amp enclosure uses 4x of the same McMaster #90666A115 for external mounting but any M4 screw could work here.

- 6x `M3x8mm` screws to hold the amplifier PCB to the internal midplate and PSU to the case. Specifically [McMaster #91290A113](https://www.mcmaster.com/91290A113/).

- 4x `M4x50` screws to secure the lid. Specifically [McMaster #91292A140](https://www.mcmaster.com/91292A140/).

And it all fits together inside an enclosure like so:

{{<figure name="cad_amp_front" >}}

{{<figure name="cad_amp_rear" >}}

## Procedure

To give you an idea of how things go together:

{{<figure name="cad_amp_01" >}}

{{<figure name="cad_amp_02" >}}

### Print

Just a few things to note:

- The suggested/ideal print orientation should be fairly obvious.
- Tolerances are reasonably tight, you'll need ~ .15mm tolerance.
  - Layer height, material, speed settings are up to you. All that matters is that you can achieve the ~.15mm tolerance.

{{< admonition warning "This will take a while!" >}}

With respect to the subwoofer skeleton and panels, at least.

Essentially, the more mass in the parts, the more energy will be required to make them rattle/flex/vibrate.
The less this happens, the less "extra" noise will be emitted by your subwoofer.
Following along with the original instructions for printing, I chose about 35% infill and 3 perimeters for the subwoofer skeleton. Using a .6mm nozzle and some other well tuned slicing settings, I was able to print the entire skeleton in about 40 hours using just over 1 kg of filament. I would plan on at least 80 if using a smaller nozzle with a finer layer height!
{{< /admonition >}}

#### Prep screw holes & flashing

{{< admonition info "Intentionally undersized holes" >}}
Almost all screw holes on this build are intentionally undersized.
As a general preparation step, **_carefully_** drive the appropriate screw into each hole and then let it sit!
{{< /admonition >}}

In the CAD model, almost all holes meant to receive a screw are intentionally undersized by about .3mm.
besides the general 'it's easier to expand the hole to the correct size than it is to shrink it' reason, this is done for one key reason; it **eliminates the need for dedicated screw thread hardware**.

By intentionally undersigning the hole, you can guarantee there will be a ton of friction generated when inserting the screws for the first time.
The excess friction _will_ soften the plastic in the vicinity of the screw threads which allows the plastic to form threads around the screw.

This is not without drawbacks, though **so proper procedure is _critical_ for making this work properly!**

You must take every precaution to ensure that:

- The screw is driven into the hole in a directly perpendicular direction.
  - As the plastic heats up, it becomes easier to knock the screw out of alignment and begin driving it at an angle.
- A constant (slow!) velocity should be used to drive the screw.
  - This can be done with hand tools but will result in a pretty tough arm workout! Consider using a power drill with a low speed / constant torque option!
- The threads of the screw will displace some plastic. The excess molten plastic is called [flashing](https://en.wikipedia.org/wiki/Flash_(manufacturing)) and it it almost guarantees that you'll need to do some post processing / cleanup of the screw holes.
- The molten plastic forms screw threads more or less instantly... but they won't _stay formed_ until the excess heat has dissipated and the material has solidified.
  - Do not remove any screws from the body until they are cool to the touch! This can take _several minutes_!

{{<figure name="sub_assembly_prep_2" >}}

After all 11 screws had cooled and the threads had formed, the screws were removed.
Nearly every one of the holes had some (small) build up of material:

{{<figure name="cut_threads_flashing" >}}

Use a straight edge blade or scrape tool to remove the flashing bits.

{{<figure name="cut_threads_flashing_bits" >}}

In some cases, there may be some additional material that builds up on the screw threads themselves.
Simply use a different screw for final assembly or use scrape tool or flame to remove the excess material from the threads:

{{<figure name="cut_threads_buildup" >}}

After the flashing is removed from around screw holes you should still be able to see threads within the hole but shouldn't be able to see/feel any protrusions _from_ the hole:

{{<figure name="cut_threads_flashing_removed" >}}

Where possible, I have made the screw holes much deeper than strictly necessary.
As the screw drives deeper, the tip will push molten plastic into this "well".

{{<figure name="cad_intentionally-deep-hole" >}}

### Subwoofer Skeleton

#### Gaskets

Begin by driving the screws into the side panels and speaker driver then attach the gaskets.

{{<figure name="sub_assembly_gasket_panel" >}}
{{<figure name="sub_assembly_gasket-speaker" >}}

{{< admonition info >}}
You do not need to press the gasket all the way up against the component; it will be compressed between the two components during final assembly.
{{< /admonition >}}

{{< admonition warning "Secure the gasketed components to the body carefully" >}}
Rotate each screw a few turns before moving to the next one. Do this [in an alternating pattern similar to how you would tighten Lug Nuts on a car](https://www.tyreplex.com/news/how-to-tighten-lug-nuts-on-a-car-the-criss-cross-pattern/). Failure to do so will likely result in the gasket being rotated out of place internally!
{{< /admonition >}}

{{<figure name="sub_assembly_gasket_pinch" >}}

#### Speaker wire

Solder ~14 inches speaker wire to the RCA jack then install in the housing.
Don't forget to use heat shrink tubing or similar to insulate/protect the electrical connections!

Pass the housing assembly through the speaker hole and the attach the driver and then housing to the back of the subwoofer.

{{<figure name="sub_assembly_rca-through" >}}

Firmly press the speaker wire into the the cable channel and push any slack wire into the RCA jack housing.

{{< admonition info >}}
There should be - at most - a few inches / 15cm of slack wire. Any more than this and you'll run out of room inside the small housing!
{{< /admonition >}}

Once the speaker and jack housing are attached to the subwoofer body, simply line up and then attach the panels with the gaskets.

Now is a good time to test that your soldering / connections are working!

If you elected to use the gasket cord material, you would need to spend a non-trivial amount of time carefully cutting and routing segments of the material to fit inside the channel.
You will almost certainly want to use superglue or similar to help you hold the material into the channel!

{{<figure name="sub_assembly_prep_1" >}}

### Amp

{{< admonition tip >}}
Just like with the subwoofer components, you'll want to ["pre-thread"](#prep-screw-holes--flashing) the various holes.
{{< /admonition >}}

#### Prep

{{<figure name="amp_rear_undersized_screws" >}}

For the various panel mounted jacks, there are small interior cavities designed to capture the nut.
This cavity is actually _oversized_ in the CAD model but due to various printer/slicing factors it is almost certainly going to be a **tight fit**!

Align the nut with the cavity and then use the screw to pull the nut into the hole then tighten the screw all the way to pull the nuts into place:

{{<figure name="amp_rear_nut_cavity" >}}

After the various holes have been prepared and the nuts have been fitted, you are ready for final assembly

#### Assemble

First, combine both parts of the midplate with the amp board. Set aside.

{{<figure name="amp_midplate" >}}

##### Power

Solder mains wires to the Powercon receptacle and terminate with crimp connectors.
Secure the Powercon receptacle to the rear of the enclosure and then attach to the screw terminals on the PSU.

Lower the PSU into the enclosure and secure it with the two screws in opposite corners.
Start with the screw closest to the Powercon connector:

{{<figure name="amp_psu_screw_2" >}}

Then add the seconds screw in the opposite corner:

{{<figure name="amp_psu_screw_1" >}}

Take this opportunity to power up and confirm that the polarity and voltage is correct!

{{<figure name="amp_mains" >}}

##### Amp PCB

Push the three rotary encoders through the holes in the enclosure and then lower the midplate/PCB on top of the PSU.
Route the DC wires up and around to the DC plug on the PCB.

{{<figure name="amp_dc_attached" >}}

Install the RCA jacks and route wires as needed to the PCB.

{{<figure name="amp_wires_done" >}}

Then connect both the USB and RJ54 jacks on the amp module to their respective panel mount jacks.
I forgot to get a picture of the USB cable but it works more or less the exact same way the ethernet cable does:

{{<figure name="amp_rear_add_eth" >}}

Either remove the BT/WiFi antenna or secure it to the inside of the enclosure just above the LEDs (not pictured here).

And before attaching the lid, attach the knobs:

{{<figure name="amp_almost_done" >}}

And just like that, the complete sub and amp.

{{<figure name="sub_and_amp_installed-feature" >}}

# Files and licenses

All the 3d printable files associated with this post are available from my [printables.com](https://www.printables.com/social/8405-rainb0w_wheez3/about) page:

- [Modified Skeleton Waveguide Subwoofer Speaker](https://www.printables.com/model/273285-modified-skeleton-waveguide-subwoofer-speaker)

- [Arylic Up2Stream Amp 2.1 enclosure](https://www.printables.com/model/273302-arylic-up2stream-amp-21-enclosure)

On that page, you will find a list of files like:

```shell
❯ tree
.
├── amp-enclosure
│   ├── amp-box-lid.step
│   ├── amp-midplate-1.step
│   ├── amp-midplate-2.step
│   ├── amp-pcb.step
│   └── amp-shell.step
└── sub
    ├── gasket-bottom.step
    ├── gasket-speaker.step
    ├── gasket-top.step
    ├── main-body.step
    ├── panel-bottom.step
    ├── panel-top.step
    ├── rca-enclosure.step
    └── sub-components v28.f3z
```

- All photos used in this post are mine and are licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

- Everything within the `amp-enclosure` is my original work. These files are licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

- Everything in the `sub` directory is based off of the fusion360 file from [zx82net](https://www.printables.com/social/107245-zx82net/about) which  is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Likewise, my modifications fall under the same license.
  - My modifications in 'source' formate are contained in the `sub-components v28.f3z` archive but to make printing easier, individual components are exported as STEP files.

## Supporting files

In building the various parts in the `amp-enclosure` directory, I relied on the following additional models:

- [Tang Band W3-1876S 3" Subwoofer](https://grabcad.com/library/tang-band-w3-1876s-3-subwoofer-1) by [George Todd](https://grabcad.com/george.todd-1)

- [Fonte Chaveada Bivolt 24Vcc 4A 100W](https://grabcad.com/library/fonte-chaveada-bivolt-24vcc-4a-100w-1) by [Eliandro Baseggio ](https://grabcad.com/eliandro.baseggio--1)

