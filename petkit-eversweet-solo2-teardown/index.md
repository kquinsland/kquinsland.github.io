# #TwoMinuteTeardown: Petkit Eversweet Solo 2

# Two Minute Teardown: Petkit Eversweet Solo 2

{{<figure name="feat-render">}}

I am doing research for an upcoming project and was curious about how other people have solved a similar problem.
[This](https://www.amazon.com/dp/B0B3RWF653) happens to be the cheapest option that was available at the time.

I am primarily interested in the pump; the integrated "low water" sensor and the UV-C sterilization are very nice features that would check off a lot of boxes for me... Unfortunately this pump it lacks both the flow rate and the head that I need for my project.

Oh well.

The pump is sealed with a potting compound so I was not able to do a full teardown.

{{<figure name="td.pump.01">}}
{{<figure name="td.pump.02">}}
{{<figure name="td.pump.03">}}

The base isn't that spectacular; a cheap micro with BTLE controlling power to a basic wireless power transmission module.
The wireless power module is covered in conformal coating so the IC number was a pain to read but it is a XKT-511.
I can find [_very_ little information](https://leap.tardate.com/electronics101/power/wirelessledmodule/) about this IC.
As far as I can tell, it is similar to Qi wireless power but without any signaling or modulation of the carrier signal.
Various qi receiver devices will briefly "wake up" when placed on the base but they do not stay powered on or charge.
My [qi tester]({{<relref "wozniak-WPX-0001A-qi-tester-teardown">}}) does power up and measures about .2W but it can't recognize the protocol.

{{<figure name="td.base.01">}}
{{<figure name="td.base.02">}}
{{<figure name="td.base.03">}}
{{<figure name="td.base.04">}}
{{<figure name="td.base.05">}}

