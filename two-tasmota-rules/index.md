# Two Tasmota rules

<!-- markdownlint-disable-file MD002 -->

[Tasmota](https://tasmota.github.io/) is an incredibly powerful alternative/open source firmware for the ever popular Espressif family of WiFi equipped microcontrollers.
This does not need to be another post espousing it's many awesome qualities, so just trust me on this; Tasmota is AWESOME.

Tasmota supports user configurable [rules](https://tasmota.github.io/docs/Rules/) which are simple commands wired into various device triggers.
In short; a device running Tasmota gains some autonomy to react to events without needing to report the event to, and wait for commands from, a remote server.

I find the rules syntax to be a bit awkward and the list of practical examples feels like autodoc with a few practical examples sprinkled in.
I have _never_ been able to craft a Tasmota rule without also consulting the list of [commands](https://tasmota.github.io/docs/Commands/)
and doing a few searches through the github issues looking for issues/questions similar to mine.

This post is a simple "here's how i did it" that I wish I had found when I needed to solve the problem.
Hopefully this will save you some trouble!

### Mr. Coffee

{{< admonition note >}}
I have since replaced my Tasmota based automation with an ESPHome based automation.
Details on that are [here]({{< relref "posts/2023/01/improved-esphome-coffee-automation" >}}).
{{< /admonition >}}

A $15 '[smart relay](https://tasmota.github.io/docs/devices/Sonoff-Pow/)' is all that's needed to turn a simple drip-over coffee maker into a remote-controllable coffee maker.
Immediately, you gain the ability to start brewing coffee in the morning from bed.
With a little extra integration work, start brewing coffee 10 minutes before your alarm is scheduled to go off. Simple quality of life improvement!

Every coffee maker has at least one safety interlock to disable the heating element if it gets too hot, but some have additional interlocks. Specifically,
my coffee maker has two more interlocks in series with the heating element; if the carafe is removed or the water tank is empty, the circuit is broken and
the heater immediately stops producing heat. From the perspective of the smart relay, the power consumption is either 0 Watts or a bit over 1100 Watts.

I was not comfortable using Home Assistant to monitor power consumption and then toggle the relay off after observing an interlock kick in.
If something happened to the WiFi connection or the MQTT server or Home Assistant, there's no way to disconnect power from the coffee maker; the relay will stay 'on'.
If the interlock failed, somehow, the heating element would immediately begin pumping out heat when it _really_ shouldn't be!

Writing a simple "if power consumption drops below 1100 Watts, turn relay off" rule would be enough, except there's no way to distinguish between the thermal interlock tripping
and the carafe being removed for a quick pour. The simple rule is incompatible with the extra interlocks; it will prematurely stop the brewing 100% of the time the carafe is removed.
If you want to pour a cup of coffee while it's still brewing, this is rather inconvenient.

Solution: give the user 30 seconds to return the carafe during brewing before turning the relay off.

From the perspective of the outlet, ignore any lulls in power consumption as long as they're shorter than 30 seconds. Otherwise, assume that an interlock _has_ tripped.

This is implemented with two rules:

```console
Rule1
    # When relay1 (heater) is turned on, activate rule2
    ON Power1#state=1 DO Rule2 1 ENDON

    # and deactivate rule2 when the heater is turned off
    ON Power1#state=0 DO Rule2 0 ENDON

    # When Timer1 expires, turn the heater off
    ON Rules#Timer=1 DO Power1 off ENDON

    # When the current used by the heating element rises above 1 Amp, disable Timer1
    ON Energy#Current>1 DO RuleTimer1 0 ENDON

    # ... and enable rule 2
    ON Energy#Current>1 DO Rule2 1 ENDON

Rule2
    # Wait for power use to drop to 0; start counting down
    ON Energy#Current<.2 DO RuleTimer1 30 ENDON

    # and disable rule2 so the timer is not constantly reset
    ON Energy#Current<.2 DO Rule2 0 ENDON
```

I chose `.2` Amps as the trigger, but the logic works the exact same way with Power / Wattage.

## Electric Kettle

I wanted to detect when my electric kettle was done boiling the water so I could play a sound and flash the lights in whichever room I happened to be in at the time.
This is trivial to do with a '[smart outlet](https://tasmota.github.io/docs/devices/Sonoff-S31/)' and a [template sensor](https://www.home-assistant.io/integrations/binary_sensor.template/) in Home Assistant:

```yaml
- platform: template
  sensors:
    kettle_running:
      friendly_name: "Kettle Boiling"
      # If the kettle is using more than 5w, assume its on / boiling water
      icon_template: >-
        {% if states('sensor.kettle_energy_power') | float > 5 %}
          mdi:kettle-steam
        {% else %}
          mdi:kettle
        {% endif %}
      value_template: "{{states('sensor.kettle_energy_power') | float > 5}}"
```

Only problem, Tasmota only publishes the kettle's power once every 300 seconds. This means `sensor.kettle_energy_power` only gets updated every 300s which makes it really hard
to fire off a _timely_ notification when the kettle is done.
The 'brute-force' solution is to configure Tasmota to transmit the telemetry [continuously](https://github.com/arendst/Tasmota/issues/2567).
Except there's no need for the Tasmota device on the kettle to be constantly informing Home Assistant that there's no power being used whenever the kettle is not on; my application only cares about the '[falling edge](https://en.wikipedia.org/wiki/Signal_edge)'.

Borrow the trigger from the [Mr. Coffee]({{< relref "#mr-coffee" >}}) rule and shorten the [telemetry period](https://tasmota.github.io/docs/Commands/#teleperiod) only when the kettle
is consuming more than 1 Watt. Like the coffee maker, it's either consuming 0 Watts or about 1.1 Kilowatt.

```console
Rule1
    on ENERGY#Power>1 do Backlog TelePeriod 10; Rule2 1 endon

Rule2
    on ENERGY#Power=0 do Backlog TelePeriod 1; Rule2 0 endon
```

As soon as the kettle starts using more than 1 Watt, configure Tasmota to publish its sensor data every 10 seconds.
I'll get notified that the water is boiling within 10s; much better than 300s!
Once the power consumption goes back down to 0, reset the telemetry update period and disable the rule.

The second rule must be disabled to prevent constant triggering of the rule when the kettle is not boiling.
Without the `Rule2 0`, the console/logs for the device would be full of statements like this repeating every few seconds:

```console
RUL: ENERGY#POWER>1 performs "TelePeriod 10"
MQT: stat/tasmota_kettle/RESULT = {"TelePeriod":10}
```

Hope that helped!

-----

**EDIT:** 2021-09-24: small typo fixes. Thanks for pointing them out :).

