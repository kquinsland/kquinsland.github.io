# Watch Band RFID fob

A compact, wearable RFID fob that can be easily attached to a watch band.

Canonical: https://karlquinsland.com/watch-band-rfid/
Published: 2024-09-14
Updated: 2024-09-14
Tags: fusion-360


---

# Watch Band RFID fob

This is a quick project to create a wearable RFID fob that can be easily attached to a watch band.
The idea is to have a small RFID tag that can be used for various purposes, such as access control or identification that is - more or less - as convenient as a surgically implanted tag, but without the need for embedding anything in your body.

I was inspired by both the [bio-hacking community](https://en.wikipedia.org/wiki/Microchip_implant_(human)) and the medical-alert 'accessory bands' for Android Wear devices.

![The inspiration for this project](https://karlquinsland.com/watch-band-rfid/images/official-medical-band-render.webp)

_The inspiration for this project_


Don't mind the few small print imperfections; _this was a prototype_ printed with a .6mm nozzle.
A final version of this would be printed with a smaller nozzle, a finer layer height and probably a different color/material.

The point is to have a small, unobtrusive design that can be easily attached to a watch band.
Because of the wide variety of watch bands available, I have [included](#files) the raw Fusion360 file for you to modify as needed.

![Installed on a watch band.](https://karlquinsland.com/watch-band-rfid/images/prototype03.webp)

_Installed on a watch band._


![A single watch band pin is used to secure the plastic part to the band.](https://karlquinsland.com/watch-band-rfid/images/prototype02.webp)

_A single watch band pin is used to secure the plastic part to the band._


![Hopefully this gives you an idea of how the parts come together.](https://karlquinsland.com/watch-band-rfid/images/prototype01.webp)

_Hopefully this gives you an idea of how the parts come together._


## Bill of Materials

The core component of this project is the [RFID tag](#sourcing-the-rfid-tag) which needs to be small and reprogrammable.

Other than that, you will need to design or print your own enclosure and secure it to the watch band with a standard pin.
Sourcing these is quite easy; a local watch shop or jeweler should have them available or you can buy a watch-band kit online.
A kit is advisable if you need to prototype multiple designs / play with different pin diameters/lengths.

Printing the tag enclosure is straightforward but due to it's small size, the more precise your printer is, the better the results will be.

![Suggested print orientation for best results](https://karlquinsland.com/watch-band-rfid/images/suggested-print-orientation.webp)

_Suggested print orientation for best results_


## Sourcing the RFID tag

This application is severely space constrained, so you need the smallest possible tag that is also reprogrammable.
If you search for tags meant to be embedded under the skin of animals, you will find many options... but they are not reprogrammable.

There are also tags meant for [humans](https://dangerousthings.com/category/implants/x-series/) but they're not cheap!
I did eventually find a suitable tag on eBay:

![eBay listing for the RFID tag](https://karlquinsland.com/watch-band-rfid/images/eBay-listing.webp)

_eBay listing for the RFID tag_


eBay being what it is, there's no guarantee that [the same listing I purchased from](https://www.ebay.com/itm/254607033331) will still be active in the future so I am including a copy of the listing description that you can use for reference when searching for a suitable tag:

> New 2pcs T5577 125KHz E/H Rewrite Writable R/W Proximity Bio Glass Tag 2.12x12mm

A portion of the listing text:

> Specifications:
>
> This is the 125KHZ T5577 Glass RFID Writeable tags Proximity Induction For animal identification or other 125KHz RFID Access controller.
>
> FEATURES:
>
> - This tag very small so difficult to read or write,please use the better device or sensor strong device.
>
> - Operation frequency: 125Khz
>
> - Chip:T5577,Rewrite Writable EM4100 & HID Chip(This Chip better and factory cost high than EM4305)
>
> - Compatible with 125KHz EM4100 or 125KHz HID reader to gain access controller and the animal identification.
>
> - Detecting distance: 0-5cm(if the Reader sensor Different so the Read Distance will Different ).
>
> - Glass Housing Material:Bio-glass.
>
> - Size:2.12 x12mm
>
> - Page included: 2pcs 125KHz T5577 Writeable Glass RFID TAGS

## Files

- [images/eBay-listing.webp](https://karlquinsland.com/watch-band-rfid/images/eBay-listing.webp)
- [files/fusion-export.step](https://karlquinsland.com/watch-band-rfid/files/fusion-export.step)
- [files/main.f3z](https://karlquinsland.com/watch-band-rfid/files/main.f3z)
- [images/official-medical-band-render.webp](https://karlquinsland.com/watch-band-rfid/images/official-medical-band-render.webp)
- [images/suggested-print-orientation.webp](https://karlquinsland.com/watch-band-rfid/images/suggested-print-orientation.webp)
- [images/prototype01.webp](https://karlquinsland.com/watch-band-rfid/images/prototype01.webp)
- [images/prototype02.webp](https://karlquinsland.com/watch-band-rfid/images/prototype02.webp)
- [images/prototype03.webp](https://karlquinsland.com/watch-band-rfid/images/prototype03.webp)


