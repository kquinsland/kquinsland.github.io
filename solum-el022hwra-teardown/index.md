# Solum 2.2 inch Electronic Shelf Label Teardown

Specifically the EL029F3WRA

Canonical: https://karlquinsland.com/solum-el022hwra-teardown/
Published: 2023-07-20
Updated: 2023-07-20
Tags: teardown


---

# Quick peek inside the EL029F3WRA

In light of the recent news that the [OpenEPaperLink](https://github.com/jjwbruijn/OpenEPaperLink) firmware has been [ported to the `nRF52811`](https://www.youtube.com/watch?v=Ph5BXFcugC4) series of chips, I decided to take a look inside the Solum EL029F3WRA to see if it was a viable candidate for a project.

As it turns out, it is very similar to the [2.9 inch](https://github.com/jjwbruijn/OpenEPaperLink/wiki/2.9%22-EL029H3WRA) tags.
I hope to soon have a simple CAD model of the ESL which I'll use to model a basic pogo-pin programming jig for this particular ESL.

-----

![The rear of the ESL is clipped in and can be removed with a plastic spudger.](https://karlquinsland.com/solum-el022hwra-teardown/images/td01.webp)

_The rear of the ESL is clipped in and can be removed with a plastic spudger._


![All programming pins are accessible from the rear once the batteries are removed.](https://karlquinsland.com/solum-el022hwra-teardown/images/td02.webp)

_All programming pins are accessible from the rear once the batteries are removed._


![Batteries are held in with moulded plastic clips.](https://karlquinsland.com/solum-el022hwra-teardown/images/td03.webp)

_Batteries are held in with moulded plastic clips._


![Side profile of the ESL.](https://karlquinsland.com/solum-el022hwra-teardown/images/td04.webp)

_Side profile of the ESL._


![The outer front panel is glued in place. Careful prying was enough to remove it.](https://karlquinsland.com/solum-el022hwra-teardown/images/td05.webp)

_The outer front panel is glued in place. Careful prying was enough to remove it._


![Unfortunately, there's more glue holding the eInk panel to the PCB and the whole PCB assembly in the case.](https://karlquinsland.com/solum-el022hwra-teardown/images/td06.webp)

_Unfortunately, there's more glue holding the eInk panel to the PCB and the whole PCB assembly in the case._


> [!WARNING] Warning
> DO NOT DO THIS.
> 
> The PCB **can** be pried out of the case but doing so will likely damage the eInk panel unless you apply pressure equally from multiple corners.
> 
> The glass on the eInk panel is very thin and far less flexible than the PCB.
> Note the crack...


![The residue is not there by default. Since I had already ruined the panel, I was testing the effect of different solvents on the panel hoping to find one that would dissolve the glue without damaging the panel.](https://karlquinsland.com/solum-el022hwra-teardown/images/td07.webp)

_The residue is not there by default. Since I had already ruined the panel, I was testing the effect of different solvents on the panel hoping to find one that would dissolve the glue without damaging the panel._


![The eInk panel is attached to the PCB with a thin layer of glue. The same tool/technique used to remove the front panel can be used to remove the eInk panel.](https://karlquinsland.com/solum-el022hwra-teardown/images/td08.webp)

_The eInk panel is attached to the PCB with a thin layer of glue. The same tool/technique used to remove the front panel can be used to remove the eInk panel._


![PCB as seen from the rear of the ESL.](https://karlquinsland.com/solum-el022hwra-teardown/images/pcb01.webp)

_PCB as seen from the rear of the ESL._

![A close up look at the micro and related pcb traces.](https://karlquinsland.com/solum-el022hwra-teardown/images/pcb02.webp)

_A close up look at the micro and related pcb traces._

![Close up look at the second half of the PCB.](https://karlquinsland.com/solum-el022hwra-teardown/images/pcb03.webp)

_Close up look at the second half of the PCB._

![Closeup of the side of the PCB normally covered by the eInk panel. Note the thin layer of glue around the edge of the PCB used to hold the eInk panel in place.](https://karlquinsland.com/solum-el022hwra-teardown/images/pcb04.webp)

_Closeup of the side of the PCB normally covered by the eInk panel. Note the thin layer of glue around the edge of the PCB used to hold the eInk panel in place._


