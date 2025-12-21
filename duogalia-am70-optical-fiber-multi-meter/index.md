# #TwoMinuteTeardown: Duogalia AM70 Optical Fiber Multi-Meter

# Two Minute Teardown: Duogalia A-M70 Optical Fiber Multi-Meter

{{<figure name="feat-render">}}

While diagnosing a flakey fiber link, i ran into a situation where a basic "flashing red" visual fault finder was not sufficient.
I needed to actually measure the optical power levels on both ends of the link to ensure that the fiber was not damaged and that the SFPs were operating within spec.

[This](https://www.amazon.com/dp/B0B6TY7X6K) was the cheapest tool that could be delivered quickly.
The issue was found and fixed so now let's have a look inside.

{{< admonition note "Many Generics" >}}
There are many many similar looking units on Ali Express and the other usual import sites.
I do not know who the ODM is but it's clear that the `Duogalia A-M70` is just another rebrand of the same design.

As best as I can tell, the ODM for this is `T-Better`?

{{< /admonition >}}

## Teardown

The nice thing about simple devices like this is that they are easy to open and rather predictable inside.
From the back, there are four self-tapping screws holding the back panel on.
Removing those reveals a very simple interior.

The few unpopulated components near where the battery attaches are almost certainly for a "better" model that has a built in battery that can be recharged over the non populated USB-C port.

The square IC in the middle is running the show, the large IC off to the side is likely the display driver and the two SOP-16 chips near the ethernet port and optical sensors are for abstracting the details away for the benefit of the microcontroller.

{{<figure name="td-02">}}

There are 4 more screws holding the PCB to the front half of the case; if you remove them, you can get a look at the side of the PCB with the screen on it... revealing that there's absolutely nothing special on the other side of the PCB.

{{<figure name="td-03">}}

And here's a close up of the silk screen on the main controller that makes me think `T-Better` is the ODM.

{{<figure name="td-04">}}

## Miscellaneous

The PCB is marked with:

```text
T-Better
MINI-OPM-N32-MAIN
V11 20250617
```

The main ICs are:

- [`HGSemi / CD4017C`](https://uploadcdn.oneyac.com/attachments/files/brand_pdf/hgsemi/2B/E2/CD4017CMTR.pdf): a 10 bit decade counter to handle the ethernet cable testing functions.

- [`NSING / NATION N32G031`](https://www.nationstech.com/uploadfile/file/20220907/1662539811646982.pdf):  Cortex®-M0 microcontroller running the show.

- [`HGSemi / CD4051B`](https://www.lcsc.com/datasheet/C507163.pdf): an 8-channel analog multiplexer to handle the optical power sensor inputs. I guess the main microcontroller doesn't have enough ADC channels on its own.

- [`TM(Shenzhen Titan Micro Elec) TM1621B / TM1621B`](https://www.lcsc.com/datasheet/C5174490.pdf): the LCD driver IC for the display.

I was not able to locate the OEM or get a digital copy of the manual that came with this thing so I scanned it myself. You can fetch it [here]({{< resource-ref "product-manual-scan" >}}).

