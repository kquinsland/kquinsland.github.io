# Two quick tricks for better HomeAssistant automations

<!-- markdownlint-disable-file MD002 MD001-->

It's hard to believe, but the venerable Home Assistant project has just celebrated it's [**7th** birthday with release `.115`](https://www.home-assistant.io/blog/2020/09/17/release-115/)! While HA does get better with each new release, it is by no means perfect :D. This post came about because of a new feature in release `.115` and the subsequent limitations of that feature!

In the course of trying to integrate Home Assistant with all the things, everybody eventually hits some major limitation or bug and is forced to find some sort of workaround. Below are a few techniques that I've collected over the years to simplify some of my Home Assistant configuration and work around some of the platforms various limitations.

To be sure, nothing below is new or groundbreaking. I am just presenting how I use them in the hopes that they end up being useful to anybody else that googles similar problems.

### (slightly) easier to maintain templates

Sometimes, you'll have many entities in a given [template](https://www.home-assistant.io/docs/configuration/templating/) and - depending on the complexity of the template - you may need to refer to each entity multiple times. This is a simple technique that allows you define the list of entities one time and refer to them as needed throughout the template in  a succinct way. This makes the template a bit cleaner / shorter which inturn makes it easier to maintain.

To illustrate the technique, I've reproduced a [template sensor](https://www.home-assistant.io/integrations/template/) that averages all of the different particulate matter readings from my [outdoor weather station]({{< relref "posts/2020/08/ws3-weather-station-pm25-sensor" >}}) into a single number. **Bonus:** this approach works around the 'only one entity' limit with the [statistics](https://www.home-assistant.io/integrations/statistics/) platform!

I want to get an average of 6 different entities:

- sensor.ws3_pm_0_3mm
- sensor.ws3_pm_0_5mm
- sensor.ws3_pm_10_0mm
- sensor.ws3_pm_1_0mm
- sensor.ws3_pm_2_5mm
- sensor.ws3_pm_5_0mm

Since each entity is nearly the same, it's far easier to only store the small portion of the entity ID that is unique and programmatically generate the full entity ID once throughout the template.

```yaml
##
# In an ideal world, I would use use the AQI metric, but the data that the WS3 exposes to me does not have the correct
#   units. AQI needs PPM but I have units in particles per Litre of air. In order to turn the unit I have into a PPM
#   I need to make some guesses about the actual chemical content of the air / know the mass of each particle. Not
#   impossible... but why bother? I can generate my own metric which wll be functionally equivalent to AQI.
#   A single number that represents the state of the air; the higher the number, the more particles in the
#   air, the less healthy the air.
#
# The 'stats' sensor platform in HA does not support multiple entity IDs, unfortunately, so I have to do it w/ a template
#   sensor.
#
# See: https://community.home-assistant.io/t/get-average-of-multiple-temperature-sensors/4050/4
##
- platform: template
  sensors:
    average_particulate_count:
      unique_id: ws3_particulate_matter_sensor_average
      friendly_name: "Average Particulate Count"
      unit_of_measurement: "particles / .1L Air"
      value_template: >-
        {## We need to use namespace() so the accumulator exists outside of the loop scope ##}
        {% set accumulator = namespace(value=0.0) %}
        {## The sensors all have the same prefix so we  store only the particulate size / suffix ##}
        {% set counts = ['0_3mm', '0_5mm', '10_0mm', '1_0mm','2_5mm', '5_0mm',] %}
        {% for ct in counts -%}
          {% set val = states("sensor.ws3_pm_"+ct) | float %}
          {% set accumulator.value = accumulator.value + val %}
        {%- endfor %}
        {## Sum up all the counts and device by the number of things and round the number ##}
        {{ (accumulator.value/(counts|length))|round(2)}}
```

### Use `input_*` in the `trigger` for an automation

The 'new' feature in `.115` that I eluded to is support for using various [`input_*` objects in the `condition` section of an automation](https://www.home-assistant.io/blog/2020/09/17/release-115/#use-input_-helpers-in-conditions). This makes it  possible to modify the parameters of an automation at run time! This was possible in the past, but would require a Rube Goldberg-esque setup chaining multiple templates/inputs/automations together; it was a mess to build and maintain!

Unfortunately, there is no support for `input_*` helpers in the *trigger* section of an automation. As of release `.115`, it is impossible to do something like this:

```yaml

- id: some-example-pipe-dream
  description: Use a input_number with a trigger block
  mode: single
  trigger:
    platform: numeric_state
    entity_id: sensor.temperature
    above: input_number.max_temp
    below: input_number.min_temp
    for:
      minutes: input_number.temp_automation_time_threshold
  action:
    # Turn the fan on
    - service: switch.turn_on
      entity_id: switch.dream-machine

```

If we can find an alternate way to trigger that automation, then we can move the comparison w/ the dynamic value to the `condition` block which brings our pipe-dream closer to reality :D. Fortunately, a significant number of automations do not require *real time* triggering so if we can 'settle' for an automation that may be delayed by a brief interval then we can use the cron-like [`time_pattern`](https://www.home-assistant.io/docs/automation/trigger/#time-pattern-trigger) trigger to launch the part of the automation that we care about. In short, changing an automation from a push based trigger to a poll based trigger.

Here is an example automation that turns a bathroom fan on if the humidity in the bathroom rises above a user-configured threshold. The humidity sensor is battery powered and reports the humidity only on changes or after several seconds (whichever is longer). This means that the automation is *already* not going to trigger in 'real time' so what's the harm in delaying it's activation by a few more seconds?

```yaml
- id: bathroom-fan-on-until-humidity-below-threshold
  description: Turns on ventilation fan once humidity gets out of hand
  mode: single
  trigger:
    - platform: time_pattern
      # Behaves just like cron
      # See: https://www.home-assistant.io/docs/automation/trigger/#time-pattern-trigger
      ##
      # Every 30 seconds
      seconds: "/30"
      # We *also* want to re-run the conditional check as soon as the value of the numeric input changes
    - platform: state
      entity_id: input_number.bath_fan_humidity_threshold
    ##
    # The approach below is the 'traditional' way that only triggers the automation the 'optimal' number of times;
    #   only when the sensor state changes.
    # This is also the only way to use the very helpful above/below + for blocks but comes at the expense of not
    #   being able to modify the conditions using the input_* helpers.
    ##
    # - platform: state
    #   entity_id: sensor.bathroom_humidity
    #   above: input_number.max_temp
    #   below: input_number.min_temp
    #   for:
    #     minutes: input_number.temp_automation_time_threshold
  condition:
    # Compare the humidity value to the 'low' value. If we're above this, we turn the fan on
    ##
    # TODO: Next month of WTF, it would be lovely if we could have templates here so quickly
    #   bang out an expression like {{(input_number.some_input + 10)}}
    ##
    - condition: numeric_state
      entity_id: sensor.bathroom_humidity
      above: input_number.bath_fan_humidity_threshold
  action:
    # Turn the fan on
    - service: switch.turn_on
      entity_id: switch.bathroom_fan
```

{{< admonition note >}}

Yes, I absolutely could've used the [`state`](https://www.home-assistant.io/docs/automation/trigger/#state-trigger) trigger to fire off the contrition check every time that a new humidity reading came in. I chose to use the 'cron' approach to illustrate how to make this technique work for two reasons:

  1. I find the `state` trigger to be useful when combined with the `above:` / `below:` and `for:` 'modifiers'. Currently, those modifiers are only supported in the `trigger` block and therefore can't be configured with `input_*`. The whole point is to have some aspects of the automation that can be adjusted at run time w/o  having to edit the automation.
  2. Some automations may not have a reliable/consistent push-based trigger event that they can use. In these cases, a regular polling approach is the only 'fall back'.

{{< /admonition >}}

