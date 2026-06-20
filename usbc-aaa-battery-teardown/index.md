# #TwoMinuteTeardown: USB-C charged AAA battery

It's an AAA battery with the charger built in! Teardown and analysis.

Canonical: https://karlquinsland.com/usbc-aaa-battery-teardown/
Published: 2026-02-01
Updated: 2026-02-01
Tags: two-minute-teardown


---

# Two Minute Teardown: USB-C charged AAA battery

First, these things have existed for quite a few years now but people still seem to be discovering them.
If you are one of todays [lucky 10,000 people to find out about them](https://xkcd.com/1053/), you're welcome.
I hope these can change your mind about rechargeable batteries; they're good, now!

One of mine started failing with intermittent output voltage so I decided to take it apart and see what was inside.

{{<figure name="images_render">}}

Technically, the one that I am taking apart is branded `MXBatt` but im keeping the title of this post generic because you can find these in many places under many different brand names and I'm sure they are all more or less the same inside.

The battery on the bottom left is a known good / still working one for comparison.
The one in parts going across the top is the one I'm tearing down.
You do not need to spend a ton of time trying to peel the label off... it's going to be a pain and isn't necessary.

Using plier, pull the positive tip part off the cell and the whole thing comes apart pretty easily.

{{<figure name="images_td01">}}

And with that, here's what's inside:

{{<figure name="images_td02">}}

{{<figure name="images_td03">}}

{{<figure name="images_td04">}}

Yep, that's all there is to it!

## ICs and Markings

The only chip on the tiny PCB is marked [`LC9201D`](https://en.eeworld.com.cn/bbs/thread-1250284-1-1.html)

From the linked page:

> The LC9201D chip is highly integrated. One chip completes the functions of charging, discharging and lithium battery protection.
> The peripheral circuit only needs two capacitors, one resistor and one inductor to realize it.

Yep, can confirm. The PCB has two capacitors, a resistor, a LED and the inductor.

Had I known what was inside beforehand, I wouldn't have bothered taking it apart... once you google the IC number, you find other people have [beaten me to the teardown](https://www.youtube.com/watch?v=skCxRBtkhLE).

## Battery Capacity

The first words out of my mouth when I saw this was "there's no way this is 1200mAh".

For reference, the grey battery cell that came out has a mass of about 3.85g.
The OD is about 9.2mm and it's about 27.5mm tall (not including the leads).

If you google the patent number on the battery, you get this [press release](https://www.huahuibattery.com/huahui-energy-wins-first-chinese-patent-award-for-large-size-monobloc-batteries.html).

I can't say for sure what the exact model/chemistry is, but if I poke around the Huahui Battery site, it looks like every 10x30mm cell they have weighs about 3.8 gram and has a capacity of about 100mAh at 3.2 - 3.7v (depending on chemistry).

Let's do a bit of math:

- Capacity: **100 mAh**
- Nominal voltage: **3.2 V**

Energy in watt-hours:

$
E = V \times Ah = 3.2 \times 0.1 = 0.32 \text{ Wh}
$

Since we know the AAA form factor is typically used in devices that expect **1.5 V** cells, let's see what the effective capacity would be  when we buck the voltage down to 1.5 V with _no losses whatsoever_.

$
Ah_{out} = \frac{E}{V_{out}} = \frac{0.32}{1.5} \approx 0.213 \text{ Ah}
$

$
\boxed{\approx 213\ \text{mAh at 1.5 V (ideal)}}
$

So ... yeah, not 1200mAh.
Let's work backwards, then.

We know the nominal voltage is 3.7v and the outer diameter must be about 10mm (to fit in AAA sized devices).
Let's assume that since we are dropping the voltage by about half, the rated 1200mAh capacity would come from a 3.7v cell at about 600mAh.

Now, how long of a cell would we need to get 600mAh at 3.7v?

$
E = 3.7 \times 0.6 = 2.22 \text{ Wh}
$

Ok, we need 2.22 Wh of energy storage.
If we assume a cell-level energy density of about 500–700 Wh/L (which is reasonable for modern Li-ion cells), we can estimate the required volume and length.

$
V = \frac{2.22}{\text{Wh/L}}
$

- 700 Wh/L → $ 2.22/700 = 0.00317\,L = 3.17\,cm^3 $
- 600 Wh/L → $ 2.22/600 = 0.00370\,L = 3.70\,cm^3 $
- 500 Wh/L → $ 2.22/500 = 0.00444\,L = 4.44\,cm^3 $

And then volume to length conversion:

$
r=5\text{ mm}=0.5\text{ cm}
$

$
A=\pi r^2=\pi(0.5)^2=0.785\,cm^2
$

$
L=\frac{V}{A}
$

| Assumed cell-level energy density | Required volume |                  Length @ 10 mm OD |
| --------------------------------: | --------------: | ---------------------------------: |
|                          700 Wh/L |        3.17 cm³ | 3.17 / 0.785 = **4.04 cm ≈ 40 mm** |
|                          600 Wh/L |        3.70 cm³ | 3.70 / 0.785 = **4.71 cm ≈ 47 mm** |
|                          500 Wh/L |        4.44 cm³ | 4.44 / 0.785 = **5.66 cm ≈ 57 mm** |

So we need something between 40 mm and 57 mm long to get 600mAh at 3.7v.
The cell inside this USB-C AAA battery is only about 27.5 mm long... 🤔.

