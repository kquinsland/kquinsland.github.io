# Adding an APC UPS to Home Assistant energy dashboard



**EDIT:** (2021-09-19): After some [back and forth](https://old.reddit.com/r/homeassistant/comments/pi3pv2/how_to_use_an_apc_ups_as_an_energy_dashboard/hbqudh5/) with [/u/Laxarus](https://old.reddit.com/user/Laxarus), there is now a simpler method! The `snmp` platform still does not support setting `device_class`, but wrapping the sensor in another template sensor is not required; just do so in your [`customize.yaml`](https://www.home-assistant.io/docs/configuration/customizing-devices/). I have called this out [below]({{< relref "#edit-2021-09-19" >}}).

-----

This is another quick "here's how I did it, hope this help" post.

In preparation for the inevitable grid brownouts that summer 2021 would bring, I installed a rather beefy UPS for my home network / lab. After some browsing, I discovered a local eWaste liquidator with a really good deal on some second-hand APC UPSs.

A few hundred dollars and about 150 lbs later, the UPS was installed in the server rack. Despite being a newer generation, the software on the UPS has a _**TON**_ of similarities to the [older style of PDU that I installed in my lab ]({{< relref "/posts/2020/12/monitoring-ap7900-switched-pdu-prometheus-grafana">}}) a while back. This made it relatively straightforward to use the same
pattern to start getting UPS metrics into Grafana as well.

After getting the basic monitoring up and running, I started to draft this post to serve as an 'update' to the APC9700 post. Life got in the way and the post sat in the drafts branch where it was completely forgotten about.... until [Home Assistant released their new Energy dashboard](https://www.home-assistant.io/blog/2021/08/04/release-20218/). Now that HA could show the energy consumption of individual devices right next to the cumulative consumption and production data, the post was worth finishing and expanding on.

The configuration that I was _going_ to publish is [below]({{<relref "#current-and-voltage-independently">}}) but after finding [this](https://old.reddit.com/r/homeassistant/comments/pi3pv2/how_to_use_an_apc_ups_as_an_energy_dashboard/hbpqzr6/) comment by [Laxarus](https://old.reddit.com/user/Laxarus) on [Navydevildoc](https://old.reddit.com/user/Navydevildoc)'s [reddit post](https://old.reddit.com/r/homeassistant/comments/pi3pv2/how_to_use_an_apc_ups_as_an_energy_dashboard/), I've got a revised and simpler configuration to share!

## Long Term Statistics in Home Assistant

Before diving into the configuration, a little bit of context. In preparation for the energy sub system, the Home Assistant developers have been working on a '[long term statistics](https://developers.home-assistant.io/docs/core/entity/sensor/#long-term-statistics)' (LTS) framework. The LTS framework is meant to give HA some improved speed and capabilities when dealing with a lot of data! The energy subsystem is the first 'consumer' of the LTS framework.

Home Assistant will look for two 'properties' on a given sensor to determine if that sensor will work with the LTS framework.
For a sensor/entity to 'work' with the long term stats system it must:

- have a property called `device_class` with a value of `energy`, `gas`, or `monetary`
- have a property called `state_class` with a value of either `measurement` or `total_increasing`

In testing, I was _not_ able to get a sensor with `state_class: measurement` and `device_class: energy` to 'work' with the energy sub system. Fortunately, this does not apply with the concise configuration [below]({{<ref "#a-single-oid-for-power-consumption">}})!

As the LTS framework is still new, many platforms - including the SNMP platform - do not support the required properties:

```log
Invalid config for [sensor.snmp]: [state_class] is an invalid option for [sensor.snmp]. Check: sensor.snmp->state_class.
Invalid config for [sensor.snmp]: [device_class] is an invalid option for [sensor.snmp]. Check: sensor.snmp->device_class.
```

The `template` platform _has_ been updated to work with the `device_class` or `state_class` properties though.
So that's the technique to use here; a `template` sensor with the correct `{device,state}_class` properties set will wrap the snmp sensor.

Hopefully a future release of HA will include `{device,state}_class` support for the `snmp` platform; the `template` sensors in the configuration snips below won't be needed!

### EDIT (2021-09-19)

You don't need to wrap the snmp sensor in a template sensor. As of home assistant 2021.09, the `snmp` platform does not allow you to set `device_class: energy`... however, you _can_ set the `device_class` attribute on the snmp sensor through `customize.yaml`:

Make sure your configuration file loads the customization file:

```shell
❯ cat configuration.yaml | grep customize
  customize: !include customize.yaml
```

If your snmp sensor was called `sensor.usp_energy`, then you would add an object called `sensor.ups_energy` like so:

```shell
❯ cat customize.yaml | grep -a2 ups
sensor.ups_energy:
  state_class: total_increasing
  device_class: energy
```

## Configure Home Assistant

I have broken my `configuration.yaml` up to make things easier to manage. Almost all entity/device/template/sensor..etc configuration is done through files placed in the `devices/**/*` directory:

```shell
❯ cat configuration.yaml | grep -E 'devices/sensor|template/'
sensor: !include_dir_merge_list devices/sensor/
template: !include_dir_merge_list devices/template/
```

### A single OID for power consumption

Thanks again to [Laxarus](https://old.reddit.com/user/Laxarus) for the [tip](https://old.reddit.com/r/homeassistant/comments/pi3pv2/how_to_use_an_apc_ups_as_an_energy_dashboard/hbpqzr6/) about the `upsHighPrecOutputEnergyUsage` OID!

First, create the 'raw' sensor using the `snmp` platform:

`devices/sensor/snmp.yaml`:

```yaml
- platform: snmp
  name: "UPS Energy (raw)"
  host: 192.168.1.1
  # The current in tenths of amperes drawn by the load on the UPS.
  # Contained in Module(s): PowerNet-MIB
  ##
  baseoid: .1.3.6.1.4.1.318.1.1.1.4.3.6.0

  # Determines whether the sensor should start and keep working even if the SNMP host is unreachable or not responding.
  # This allows the sensor to be initialized properly even if, for example, your printer is not on when you start Home Assistant.
  accept_errors: true

  # UPS reports in tens of kWh so we'll need to divide by 10 to get kWh; HA only accepts kWh or Wh for sensors
  #   that will 'work' on the energy dashboard
  unit_of_measurement: kWh
  value_template: "{{ ((value | int) / 10) | float}}"
```

Then, wrap the `snmp` sensor with the necessary properties:

`devices/template/ups_energy.yaml`:

```yaml
- sensor:
    - name: "UPS Energy"
      # Unique ID is required for mgmt through the web UI
      unique_id: tmpl-ups-energy
      icon: mdi:lightning-bolt
      # Required for Energy dashboard
      state_class: total_increasing
      device_class: energy
      unit_of_measurement: kWh
      state: "{{ (states('sensor.ups_energy_raw') | float) }}"
```

Restart Home Assistant and you should now be able to add `sensor.ups_energy` to the list of individual devices on your Energy Dashboard :D.

### Current and Voltage independently

After sorting through the _massive_ MIB file that APC publishes; I only found ways to measure the voltage and current via SNMP. I assumed that APC meant for you to calculate the power use on your own from the voltage and current.

As it turns out, APC has a `upsHighPrecOutputEnergyUsage` field which reports: `The output energy usage of the UPS in hundredths of kWh.` If your APC device publishes a value on the OID `.1.3.6.1.4.1.318.1.1.1.4.3.6.0` then you can skip the configuration below; use the more concise configuration [above]({{<relref "#a-single-oid-for-power-consumption">}}). If your devices _**does not**_ publish the cumulative energy consumption, it can still be calculated manually.

The manual approach below is a lot like the concise approach above; uses two `snmp` sensors to collect the voltage and current from the UPS and then wrap everything in a LTS-compatible `template` sensor to get the data to show up on the energy dashboard.

`devices/sensor/snmp.yaml`:

```yaml
# Unfortunately, there is no direct 'watt' field. We need to calculate this on our own
# P = IV so if we can get the current and voltage, we can figure out the power
##
- platform: snmp
  name: "UPS Output Current"
  host: 192.168.1.1
  # The current in tenths of amperes drawn by the load on the UPS.
  # Contained in Module(s): PowerNet-MIB
  ##
  baseoid: .1.3.6.1.4.1.318.1.1.1.4.3.4.0

  # Determines whether the sensor should start and keep working even if the SNMP host is unreachable or not responding.
  # This allows the sensor to be initialized properly even if, for example, your printer is not on when you start Home Assistant.
  accept_errors: true
  # For reasons that I don't understand... HA does not like it when i specify 'device_class' :/
  #device_class: current
  unit_of_measurement: "A"
  # Because the number is in 10ths of amps, we need to shift the decimal by 1 place
  #   43 becomes 4.3
  ##
  value_template: "{{((value | int) / 10) | float}}"

- platform: snmp
  name: "UPS Output Voltage"
  host: 192.168.1.1
  #The output voltage of the UPS system in tenths of VAC.
  # Contained in Module(s): PowerNet-MIB
  ##
  baseoid: .1.3.6.1.4.1.318.1.1.1.4.3.1.0

  # Determines whether the sensor should start and keep working even if the SNMP host is unreachable or not responding.
  # This allows the sensor to be initialized properly even if, for example, your printer is not on when you start Home Assistant.
  accept_errors: true
  unit_of_measurement: "V"
  # Because the number is in 10ths of volts, we need to shift the decimal by 1 place
  #   1211 becomes 121.1
  ##
  value_template: "{{((value | int) / 10) | float}}"

```

`devices/template/ups_energy.yaml`:

```yaml
# The docs around the long term stats support for a sensor are not super clear and seem to be a bit contradictory.
# Both the power and current sensors are 'point in time' sensors and DO NOT represent an 'always increasing' value.
# The docs seem to imply that HA will do the integration for you if the sensor has `state_class` set to `measurement` or
#   `total_increasing`. But elsewhere in the docs, you have this:
#         Home Assistant tracks the min, max and mean value during the statistics period.
#         The state_class property must be set to measurement, and the device_class must not be either of `energy`, `gas`, or `monetary`.
#
# In testing, I was only able to get sensors that had `total_increasing` and `energy` set to show up / work with the energy dashboard. It looks like a `measurement` sensor _could_ be used... but only if the `last_reset` property can be set to 0... and currently this can't be done via YAML.
#
# So we lie about the sensor and tell HA that it's a `total_increasing` sensor and we just hope that the value never drops more than 10% which appears to be the signal to HA that the meter has been reset :/
#
# https://developers.home-assistant.io/docs/core/entity/sensor/#long-term-statistics
# https://www.home-assistant.io/integrations/sensor/
##
- sensor:
    - name: "UPS Power"
      # Unique ID is required for mgmt through the web UI
      unique_id: tmpl-UPS-power-use
      icon: mdi:lightning-bolt

      # This sensor records the instantaneous power load on the UPS... it is a measurement at that point in time
      state_class: measurement
      device_class: power
      unit_of_measurement: W
      state: >
        {% set current = states('sensor.UPS_output_current') | float %}
        {% set voltage = states('sensor.UPS_output_voltage') | float %}
        {{ (current * voltage) | round (2) }}

- sensor:
    - name: "UPS Energy"
      # Unique ID is required for mgmt through the web UI
      unique_id: tmpl-UPS-energy
      icon: mdi:lightning-bolt
      # Lie to HA about the type of sensor so we get values in the dashboard.
      state_class: total_increasing
      device_class: energy
      # Since we are pretending to integrate over time, we are in Wh now.
      # Good thing, too... because the energy dashboard WILL NOT use any sensor that does not report in kWh or Wh
      unit_of_measurement: Wh
      state: "{{ (states('sensor.UPS_power') | float) }}"
      # or, for kWh
      #state: "{{ (states('sensor.UPS_power') | float)/1000 }}"

```

