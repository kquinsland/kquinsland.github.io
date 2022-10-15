# I made a thing: GPS/PPS clock source for ntpd



{{< admonition tip >}}
This is just a 'pointer' post.

All the details are in the [`kquinsland/yet-another-gps-pps-opnsense` repo](https://github.com/kquinsland/yet-another-gps-pps-opnsense) on github.
{{< /admonition >}}

---

For the longest time, I had a dedicated raspberry pi with a GPS module acting as the `ntp` server for my home network. I chose to use a dedicated host for this because my router - at the time - did not have a serial port that I could leverage.

Recently, the miroSD card on the ntp pi died and I decided to leverage the serial port in my  current router rather than re-build the operating system on the ntp host.

It didn't take that much time to but together a bare-bones schematic in KiCad.
One LCSC order, one mouser order and about a week later, I had a basic RS232 <-> TTL converter for use with a cheap GPS module

{{<figure name="done">}}

After a bit more testing, I was convinced I hadn't made any egregious mistakes and the new module didn't pose a significant threat/risk to my production router.

{{<figure name="installed-feature">}}

