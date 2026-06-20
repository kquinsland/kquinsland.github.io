# Enhanced Home Assistant Switch Plate (HASP)

A few new hardware features and software tweaks to make the wonderful HASP project even better

Canonical: https://karlquinsland.com/enhanced-homeassistantswitchplate/
Published: 2020-10-01
Updated: 2020-10-01
Tags: fusion-360, has:download, home assistant


---

The [HASwitchPlate](https://github.com/aderusha/HASwitchPlate/) project by `aderusha` is brilliant. He's managed to arrange some relatively cheap commodity hardware into a package that conveniently fits into a prime location for interacting with Home Automation - the light switch. The entire package sips power off of the already present mains wiring and connects to any MQTT broker via the esp8266 chip. As the HASP was designed to be used with Home Assistant, the humble 2.4 inch LCD transforms into an accessible control surface for an incredibly powerful home automation platform!

## Modifications

First, a brief word about indemnification:

> All instructions and information on this page is purely for informational purposes.
>
> The original HASP and my modifications come with neither an expressed or implied warranty. By constructing your own, you assume all risk.

### Software

The few software modifications I've made to the HASP project are relatively small and don't implement anything significant or new; they are 'quality of life' improvements that target small portions of the code.

I strongly believe that the hardest part of using a given piece automation technology should be taking it out of the box. To wit, Home Assistant has robust - [but not perfect](https://community.home-assistant.io/t/ability-to-set-entity-icon-through-mqtt-discovery/224538) - support for [automatic device discovery and configuration via MQTT](Https://www.home-assistant.io/docs/mqtt/discovery/). Once a device has connected to the MQTT broker, it simply publishes a document detailing how HA should interface with the device. It's a wonderful experience to see a new device connect to the network for the first time and just a few seconds later the new device is registered in HA, waiting for you to start integrating it into automations 🤩🤤.

While there are quite a few sensors/stateful-attributes in the original HASP code, none of them automatically present themselves to Home Assistant 😔. I set about fixing that to the best of my abilities. Unfortunately, the LCD backlight is the only peripheral that could be 'upgraded' to automatic configuration without making a ton of changes to the original code base.

All of the hardware modifications detailed below automatically configure themselves with Home Assistant.

The original HASP code base uses allocates too little memory for storing MQTT credentials. Any username or password longer than 31 bytes/characters would be **silently** truncated on save. Despite the 'credentials saved, rebooting' message, the next screen was always a 'failure to connect' message. Frustrating! The pull request to address this is [here](https://github.com/aderusha/HASwitchPlate/pull/117).

### Hardware

The HASP does support dimming the LCD, but it has no awareness of ambient light levels. This makes it rather difficult to deploy a HASP in any 'light sensitive' location like a bedroom or media room without another device to cue Home Assistant and lower the LCD brightness. Workable, but too high friction to be ideal.

Light Dependent Resistors - [LDR](https://en.wikipedia.org/wiki/Photoresistor) for short - are a very cheap and compact way to get an approximate light level. Fortunately, the analogue pin is broken out on the [PCB](https://github.com/aderusha/HASwitchPlate/tree/master/PCB) so a simple [resistive divider](https://en.wikipedia.org/wiki/Voltage_divider#Resistive_divider) and a few lines of code are all that's needed to get a a number that linearly correlates to the level of light hitting the face plate.

It just so happens that there is _just enough_ room to fit a LDR beneath the surface of the decor plate and out of the way of the LCD. As (not) shown in the picture above, there is no way to tell that the LDR is embedded beneath the surface.

The range of values from the LDR isn't great, but they are adequate enough to reliably detect if the HASP is in a darkened room.

![Screenshot depicting graph of LDR values, rising over time. Annotated to indicate sharp rise in brightness right around sunrise](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/ldr-graph.webp)

_ignore the spikes just before noon, those are due to to testing with a flash light_


While researching potential ways to measure the ambient brightness, I stumbled across the [HomeSeer HS-WD200+](https://shop.homeseer.com/products/z-wave-dimmer-switch) which features a rather novel way to convey additional information to a user; seven RGB leds 😍. It's a brilliant use of such a simple and ubiquitous technology that I'm not really sure why every smart switch doesn't do this!

![Picture showing tools/materials required for assembly](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/HS-WD200-Modes.webp)

_HomeSeer's WS200 has a delightful innovation... RGB LEDs!_


With _zero_ room to spare, four LEDs can fit into a small cavity between the LCD and decor plate surface. Like with the LDR, there is no visual 'tell' that there are LEDs beneath the surface... unless they're on, of course.

![Picture showing a prototype HASP with LEDs embedded beneath the surface and powered on](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/prototype-leds.webp)

_Ignore the dirt on the switch. Notice how you can't tell there's an LDR beneath the surface_


#### The enclosure

Without consuming more than 1 [gang](https://diy.stackexchange.com/questions/11654/what-does-1-gang-2-gang-and-so-forth-mean-when-talking-about-electrical-bo), the only realistic way to get the wires for the new components to the HASP PCB is to go around the edge of the LCD's PCB. This is accomplished with two small 'channels' that slightly protrude in the vertical directions from the lip around the LCD and into the box that encloses the electronics.

![Screenshot of Fusion360, showing side profile of HASP, annotated to indicate channels in enclosure for wires leading to LED tape, LDR in relation to LCD](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/CAD-channel-profile.webp)

_Section analysis from f360 showing small channels around the LCD for wires to LDR and LEDs_


Additionally, I found that the original printed parts do not properly fit in the [two gang electrical box](https://www.amazon.com/Thomas-Betts-B232A-UPC-Carlon-Bulk/dp/B000GAWYZ8/) where I intended to deploy the HASP.

![Closeup photograph of prototype HASP enclosure and 2-gang electric box. Annotated to show interfering surfaces](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/electric-box-clearance.webp)

_The only way to fit the HASP in my 2-gang electric boxes is to cut out a small section_


In total, the modifications required to accommodate the LDR and LED tape are relatively small, but [I had to re-create the model from scratch](https://github.com/aderusha/HASwitchPlate/issues/116) so several things are likely a bit different from the original model. It should be relatively easy to port the modifications to the other multi-gang configurations that the original HASP supports.

The `.step` and `.f3z` files are available should you want to make the necessary modifications to other switch plate configurations.
The `.stl` and `.3mf` files are available for easy printing.

All files available [below](#files) and on [github](https://github.com/kquinsland/HASwitchPlate/tree/master/3D_Printable_Models/enhanced-HASP).

## Assembly

The steps below are meant to be followed between the original steps [`03_Electronics_Assembly`](https://github.com/aderusha/HASwitchPlate/blob/master/Documentation/03_Electronics_Assembly.md) and [`04_Project_Enclosure`](https://github.com/aderusha/HASwitchPlate/blob/master/Documentation/04_Project_Enclosure.md).

In short, you'll want to assemble and test the HASP PCB as in the original instructions, but use the steps below as a substitute for the enclosure assembly process.

### Additional Materials

In addition to everything in the original BOM, you'll need:

- A LDR. Be aware that LDRs come in many different shapes and sizes and have different resistance values. Space is rather tight, so the form factor matters a lot more than the resistance values or band of light that the LDR responds to. If you use a different LDR, you may need to adjust the resistor value to better match your LDR. You can get 50 of the exact model GL5516 that I used for less than $2 [here](https://s.click.aliexpress.com/e/_dUjV8Rn).

- A 10KΩ resistor to form a simple resistive divider with the LDR. This will better map the the full range of the LDR to the 0-1 volt range that the ESP8266 ADC uses. Space is rather limited so the smaller the resistor, the better; no need for anything larger than 1/4 Watt. You can get a 10-pack of 10KΩ resistors [here](https://rover.ebay.com/rover/1/711-53200-19255-0/1?mpre=https%3A%2F%2Fwww.ebay.com%2Fitm%2F10-Pcs-1-8-Watt-1-Metal-Film-Resistor-Choose-you-own-values-USA-SELLER%2F302971259661&campid=5338745533&toolid=10001&customid=
)

- WS2812B LED tape **featuring the 3535SMD package**. The 'regular size' LED package _will **not** fit_ so make sure you use the smaller `3535SMD` package! Beyond that, aim for LED tape that is no wider than ~8mm and has highest density possible. After quite some searching, the best that I could find is 8mm wide tape with a density of 144 LEDs per meter. Unfortunately, they don't sell LEDs 4 at a time, so you can pick up 1 meter of the exact LEDs I used [here](https://rover.ebay.com/rover/1/711-53200-19255-0/1?mpre=https%3A%2F%2Fwww.ebay.com%2Fitm%2FNarrow-DC-5V-WS2812B-Led-Strip-Individually-Addressable-WS2812-Led-pixel-Light%2F254647279915%3Fhash%3Ditem3b4a29212b%3Ag%3A9D8AAOSwIZ5fBTn7&campid=5338745533&toolid=10001&customid=pixel_tape).

- Several centimeters of very fine gauge wire. Ideally, multiple colors so you can keep organized but this is not required. Again, space is rather tight, so AWG 28 or 30 is ideal. Much bigger than that and you'll likely have issues fitting everything together. You can get a 1m length of multiple colors [here](https://rover.ebay.com/rover/1/711-53200-19255-0/1?mpre=https%3A%2F%2Fwww.ebay.com%2Fitm%2FNarrow-DC-5V-WS2812B-Led-Strip-Individually-Addressable-WS2812-Led-pixel-Light%2F254647279915%3Fhash%3Ditem3b4a29212b%3Ag%3A9D8AAOSwIZ5fBTn7&campid=5338745533&toolid=10001).

- HeatShrink Tubing (HST). The exact diameter needed will depend on how compact your soldering is and the size of resistor used. Multiple colors will help keep things organized, but that's not required. I suggest a kit so you can get the  perfect diameter  and length. You can get a kit [here](https://rover.ebay.com/rover/1/711-53200-19255-0/1?mpre=https%3A%2F%2Fwww.ebay.com%2Fitm%2F360pcs-2-1-Heat-Shrink-Tube-Tubing-Sleeving-Wrap-Wire-cable-Insulated-Assorted%2F224122863249%3Fhash%3Ditem342ec37e91%3Ag%3AMbcAAOSwXGhfPCNK&campid=5338745533&toolid=10001).

- Standard 2.54mm pin headers. You can get a set of male and female headers [here](https://rover.ebay.com/rover/1/711-53200-19255-0/1?mpre=https%3A%2F%2Fwww.ebay.com%2Fitm%2F10pcs-Male-Header-1x40-2-54mm-40-Pin-PCB-Through-Hole-Arduino-and-Pi%2F223105297271%3Fepid%3D28022683725%26hash%3Ditem33f21cab77%3Ag%3A~X8AAOSwvFpbdZWA&campid=5338745533&toolid=10001).

![Picture showing the additional electronic components for mod](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/00.prep-electronics.webp)

_All the components needed for assembly_


**Note**: Where possible, I've used commission links. The links don't raise the price of any item, they simply let the retailer know that I'm the reason for your business. In return, I get a small cut of your purchase as a 'thanks' from the retailer. If you're not comfortable with that, you can use a URL unwinding service to get the 'raw' product link and drop the attribution/commission part.

### Brass inserts

The original instructions suggest using glue to secure the brass inserts to the face plate, but I suggest using either a lighter or a very fine tip soldering iron to heat up the brass insert. A hot insert will melt the surrounding plastic which will make for a much stronger joint.  The modified version of the HASP face plate used in this guide intentionally features holes slightly too small for the brass inserts so glue won't be enough.

Fully insert the 20mm screw into the brass insert, use pliers to grasp the screw and apply heat with a lighter for a few seconds and then insert into the face plate. Make sure that the screw is **completely** inserted into the insert so that molten plastic can't flow into the insert. If molten plastic enters and cools on the threads, it will be rather difficult to later screw things together!

![Picture showing a 20mm screw fully inserted into brass threaded insert](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/01.prep-brass-inserts.webp)

_Molten plastic can't flow into a filled insert_


As the inserts cool, you'll have a few seconds to make adjustments to their orientation before they become fixed. Make sure that each screw is vertical and cool to the touch before moving on to the wiring harness.

![Picture showing threaded inserts inserted into faceplate with screws sticking out, pointed vertically](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/02.inserts-cooling.webp)

_screws make it easy to perform final alignment on inserts and act as a heatsink_


### Wire Harness

There are only six wires for the two components and the LED tape is the only component where polarity matters. A table mapping each pin on the HASP PCB to each component lead is below.

| HASP PCB Pin | Component                    |
| ------------ | ---------------------------- |
| GND          | LED Tape, 10KΩ resistor      |
| +5V          | LED Tape VCC in              |
| +3.3V        | LDR                          |
| A0           | LDR & 10KΩ resistor junction |
| D0           | N/A                          |
| D1           | LED Tape CLK/DTA  in         |
| D2           | N/A                          |
| DBG          | N/A                          |

You are **strongly** encouraged to test / check your work after each step with a simple multimeter. Complete assembly involves irreversible steps, like shrinking HST. If you make a mistake, it's best to catch it before 'finalizing' things as you're almost certainly going to have to destroy the component and or wiring harness to repair it.

Additionally, you can flash my [modified HASP firmware](https://github.com/kquinsland/HASwitchPlate) to a fully assembled HASP device and use that as a testing device. On boot up, the modified firmware briefly flashes each LED in sequence. About every 10 seconds after boot, the LDR value will be read and published to MQTT. Cover the LDR with your hand and check your MQTT broker or HA to confirm that the value changes significantly between exposed and covered states.

#### LDR

Fit the LDR into the face plate and position the leads in the channel under and to the top of the LCD PCB. Then trim the excess length off of the LDR leads so that they extend 'beyond' the LCD by about 8mm or so.

![Picture showing the LDR in position under the LCD with it's leads trimmed slightly](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/03.ldr-position.webp)

_LDR under the LCD w/ trimmed leads..._


Remove the LDR from the face plate and straighten the leads out. Lay the LDR, resistor, and wire out as shown below. Use this time to cut the HST down to size so everything but the leg of the LDR that joins the resistor can be covered. **Note:** the leads on the resistor have also been trimmed down to ~15mm to better fit.

![Picture showing rough layout of LDR and related components](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/04.wire-harness.webp)

_rough layout of LDR harness_


It's probably going to be easiest to start with the 'supply' wire which runs directly from the HASP PCB's +3.3v pin to the LDR. As space is exceedingly tight, there is no room for overly large solder joints. Carefully wrap the wire around the lead and apply just enough solder to make good electrical contact. Once the supply wire has been soldered to the LDR, affix the HST so the entirety of the lead and solder joint are protected.

![picture showing wire wrapped around the lead of LDR prior to soldering](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/05.ldr-wire-wrap.webp)

_strip ~1cm off the end of the wire, wrap around the lead, then solder_


Repeat the same procedure with the lead of the resistor that will go directly to the ground. Affix the HST after the solder joint has cooled.

![picture showing sire soldered to the 'ground-facing' lead of the 10k resistor with heat shrink tubing about to be applied](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/06.ldr-ground.webp)

_No room for mechanically strong solder joints_


Use the same 'wrap' technique with the remaining leads from the LDR and the resistor. Apply just enough solder for good electrical contact between the signal wire and the two leads of the LDR and resistor.

![picture showing wire wrapped around the LDR and 10K resistor leads](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/07.ldr-signal.webp)

_same wrapping technique, just with two leads this time_


After the solder joint cools, cover the entire LDR/Resistor assembly in HST so there is no exposed wire or component lead.

![Picture showing rough layout of LDR and related components](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/08.hst.webp)

_get everything nicely protected behind HST_


Set the LDR component aside for now and move on to the LED tape.

#### LEDs

If you've not already trimmed the LED tape to fit in the face plate, do so now. Leave yourself as much room as needed to solder on the one side where you'll need to feed power and data in.

The LED tape has incredibly small pads so take your time on this. You have about 5 cubic millimeters _total_ for the three solder joints on the LED tape. Solder just enough for electrical contact, there will be no significant mechanical stress. **Note:** The arrows printed on the strip point in the direction of data flow; combined with the off-center placement of the LED packages, there is only one way to properly install the LED tape in  the decor plate.

![picture showing wire lined up with one solder pad on the LED tape ](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/09.wire-alignment.webp)

_know which way the LED tape wires will need to point to make it into the 'channel' prior to soldering_


After soldering the power, signal and ground wires, flatten them and gently shape them so they align and flow into the channel on the face plate in a manner similar to the LDR. For a bit of mechanical robustness, consider braiding the wires together.

![picture showing three wires from LED tape carefully braised together for a more compact and stronger cable](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/10.twisted-cable.webp)

_twist the wires together for a more compact and robust cable_


#### Pin Header

After soldering wires onto the two components, all that's left is soldering the wires to the correct pins on the pin header.

One at a time, carefully take the pins out of the plastic 'frame'. This is to prevent the heat from the soldering and HST steps from melting the plastic. Consider marking the pin with the location of the 'frame' so you don't take up too much room with the solder joint. You will need the 'business end' of the pin to make good contact with the socket on the HASP board; this is quite difficult if most of the pin can't be slid through the 'frame' because of an excessively large solder joint!

![picture showing single pin removed from pin header, about to be soldered](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/11.process.webp)

_remove the pin from the plastic 'frame' and carefully solder the wires to the pins_


Make sure that HST is on the wire(s) _before_ soldering! After the solder joint cools, slip the HST over the solder joint and apply heat until the joint is covered. Repeat this process for the remaining pins. There will be three 'unused' pins that you can just ignore.

![picture showing the unused pins in the wire harness](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/12.unused-pins.webp)

_A few pins are not needed_


When all the wires have been soldered and covered,  you'll have something that looks like this:

![picture showing each component in the decor plate and the finished wiring harness.](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/13.harness-finished.webp)


Double check electrical connections and then combine the PCB, enclosure, plate and harness. Carefully align the 'channels' for LDR and LED wires:

![picture of finished assembly ready to be screwed together. Annotated to call attention to pinch points w/ wires](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/14.finished.webp)

_Align and ensure no pinched wires!_


### Printing

In the electrical boxes there's absolutely no room to spare, so all parts are designed to fit together with **very** tight clearances. Print as slow as you need to in order too achieve very good dimensional accuracy. Like with the brass inserts, the various screw holes are intentionally undersized for better thread engagement.

![Screenshot showing modified enclosure files in slicer for 3d printer](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/suggested-print.webp)

_The obvious orientation is the best, no supports needed_


The HASP does run a bit warm, but not any warmer than other smart switches with similar form-factor. PLA will likely be fine unless you in tend to deploy the HASP in a particularly hot location.

#### Thermals

In testing, the parts of the HASP ran ~20º above ambient. Most of that heat comes from the small AC->DC converter and the LCD. Surprisingly, the ESP8266 does not emit that much heat. Overall, nothing particularly alarming, especially when the HASP is deployed into a larger multi-gang electrical box with with plenty of air to radiate heat into. Apologies in advance for the mixed ºC/ºF units.

![photo from a thermal camera showing the HASP device removed from it's electrical box, the sides and rear are warm](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/01.removed-side-rear.webp)

_HASP removed from electrical box, heat from the AC-DC converter radiating through the enclosure_

![photo from a thermal camera showing the HASP device removed from it's electrical box, the sides and rear are warm](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/02.removed-top-side.webp)

_similar to first picture, parts inside enclosure radiate heat through the enclosure, bringing it to about 20 degrees above ambient temperature_

![photo from a thermal camera showing the HASP device removed from it's electrical box, the sides and rear are warm](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/03.removed-rear-side.webp)

_there are clear hot-spots on the enclosure that match up with the location of the ESP and the AC-DC converter_

![photo from a thermal camera showing the HASP device installed in its electrical enclosure. The LCD and LEDs are the warmest part](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/04.front-leds-on.webp)

_Less heat is radiated through the front of the HASP; the LCD backlight and embedded LEDs are clearly the warmest components_


The tiny PNP transistor that toggles the LCD power is the hottest component, by far. Everything else has a relatively consistent temperature:

![photo from a thermal camera showing the internals of deployed HASP; the AC->DC converter is clearly the warmest object](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/05.internals.webp)

_PNP Transistor is clearly warmest internal component, ESP8266 cooler than expected_


When installed in wall, the HASP is radiating heat into the electrical box and through neighboring switches:

![photo from a thermal camera showing the HASP as deployed in wall. Heat is evenly radiated through LCD and through neighboring switch](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/06.installed-markup.webp)

_a little bit of ambient heat radiating through the gap between LCD and switch as well as through the switch_


## Future work

I did accomplish my two main goals:

1. Ambient light sensor for more responsive backlight control
2. HomeSeer-like LEDs for additional information display

However, there's a few aspects of the HASP that i'd like to further improve:

- Increased automatic configuration w/ the HASP. There's still a non-trivial amount of work required to set up a HASP with Home Assistant. The HASP device has many different attributes that could be exposed to Home Assistant automatically. E.G.: The HASP will periodically check for a firmware update; the HASP should configure Home Assistant with a [`binary_sensor`](https://www.home-assistant.io/integrations/binary_sensor.mqtt/#configuration) that indicates if there's a firmware update available.

- Some of the automations that live in Home Assistant to manage button presses / update screen content could  probably be re-factored down to a few generic scripts and services.

- The display uses a [resistive matrix](https://forum.digikey.com/t/resistive-touch-vs-capacitive-touch-whats-the-difference/1063) to detect touch events. Resistive screens, while cheap, are a bit finnicky and inferior to the more consistent and precise capacitive touch screen tech. If Nextion ever makes a capacitive version of the NX3224T024, then it would be an instant upgrade!

- Similarly, the viewing angle on the LCD is... not great. The HASP, like most light switches, is at a convenient height for hands... not eyes. If you're standing relatively close to the HASP, you'll be looking down at the screen with quite a steep angle. Depending on the current screen brightness and colors, this may result in a washed out and illegible screen.

- Better LED diffusion. There's not a lot of space for light pipes or similar, but I would like to find a better way of diffusing the LEDs. Or even to fit a masking layer between the LEDs and the plate, so the effect would be akin to a back-lit silhouette.

- Effects for the LEDs. Right now, the LEDs only support on/off/brightness commands. Their ability to passively indicate information would be enhanced if Home Assistant could send 'effect' commands, too. E.G.: Rather than HA having to send a constant stream of color/brightness commands to fade between blue, green, yellow, purple, simply send a `fade(2s) blue, green, yellow, purple` command.

- There's plenty of spare room on the HASP PCB for the 10KΩ resistor. This would simplify the creation of the wiring harness. I've been meaning to learn KiKad, this might be a very gentle way to start learning it. 🤔

- Enclose the LED tape 100% inside the HASP. This version works well and the small 'hole' in the enclosure. I don't think there's any safety concern w/ the current design, but I'd still like to have the low voltage stuff 100% blocked off from thee electrical box...

## Files

For archive / posterity, all source files are included here. Unless otherwise explicitly stated otherwise, all files below are licensed as [Creative Commons Attribution-NonCommercial-ShareAlike](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA).

- [images/CAD-channel-profile.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/CAD-channel-profile.webp)
- [images/electric-box-clearance.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/electric-box-clearance.webp)
- [images/assembly/00.prep-electronics.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/00.prep-electronics.webp)
- [images/assembly/01.prep-brass-inserts.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/01.prep-brass-inserts.webp)
- [images/assembly/02.inserts-cooling.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/02.inserts-cooling.webp)
- [images/assembly/03.ldr-position.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/03.ldr-position.webp)
- [images/assembly/04.wire-harness.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/04.wire-harness.webp)
- [images/assembly/05.ldr-wire-wrap.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/05.ldr-wire-wrap.webp)
- [images/assembly/06.ldr-ground.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/06.ldr-ground.webp)
- [images/assembly/07.ldr-signal.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/07.ldr-signal.webp)
- [images/assembly/08.hst.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/08.hst.webp)
- [images/assembly/09.wire-alignment.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/09.wire-alignment.webp)
- [images/assembly/10.twisted-cable.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/10.twisted-cable.webp)
- [images/assembly/11.process.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/11.process.webp)
- [images/assembly/12.unused-pins.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/12.unused-pins.webp)
- [images/assembly/13.harness-finished.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/13.harness-finished.webp)
- [images/assembly/14.finished.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/assembly/14.finished.webp)
- [images/installed.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/installed.webp)
- [images/ldr-graph.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/ldr-graph.webp)
- [models/box.step](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/box.step)
- [models/box.stl](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/box.stl)
- [models/combined.3mf](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/combined.3mf)
- [models/combined.step](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/combined.step)
- [models/enhanced-HASP.f3z](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/enhanced-HASP.f3z)
- [models/plate.step](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/plate.step)
- [models/plate.stl](https://karlquinsland.com/enhanced-homeassistantswitchplate/models/plate.stl)
- [images/prototype-leds.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/prototype-leds.webp)
- [images/suggested-print.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/suggested-print.webp)
- [images/thermals/01.removed-side-rear.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/01.removed-side-rear.webp)
- [images/thermals/02.removed-top-side.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/02.removed-top-side.webp)
- [images/thermals/03.removed-rear-side.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/03.removed-rear-side.webp)
- [images/thermals/04.front-leds-on.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/04.front-leds-on.webp)
- [images/thermals/05.internals.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/05.internals.webp)
- [images/thermals/06.installed-markup.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/thermals/06.installed-markup.webp)
- [images/HS-WD200-Modes.webp](https://karlquinsland.com/enhanced-homeassistantswitchplate/images/HS-WD200-Modes.webp)


