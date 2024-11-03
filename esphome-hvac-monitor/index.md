# Monitoring HVAC system with ESPHome

# Using ESPHome to Monitor HVAC System

Here's a quick "show-and-tell" about a recent project I completed.

## The concept

I'm not sure how I first came across the idea of measuring air pressure drop across a filter to gauge its remaining life, but I remember thinking it was a great idea.

{{<figure name="concept">}}

The idea has been lurking in the back of my mind for a while, but I never got around to implementing it... until now.

Years ago, I added a [CO₂ sensor](https://www.winsen-sensor.com/sensors/co2-sensor/mh-z16.html) to my HVAC return duct to monitor indoor air quality.
While the sensor provided useful data, I was never really satisfied with how quickly it was installed (read: hacked together messy prototype), and I always wanted to improve it.

While browsing AliExpress for an unrelated project, I stumbled upon some differential air pressure probes and decided to buy one of these [sensors](https://www.aliexpress.us/item/3256807382388280.html).

After an afternoon of work, I had the sensor up and running and could [communicate with it using some simple Python](https://github.com/kquinsland/qdx50d-air-pressure-sensor-poc).

I then designed a PCB to integrate the sensors and electronics and 3D-printed a case to house the whole setup.

## The Hardware

{{< admonition note >}}

I'm not sharing the PCB or other mechanical files because they're specific to my setup.
Unless you have the same HVAC system configuration, layout, and dimensional constraints, these files likely won't be useful.
This post aims to share the concept and hopefully inspire others.
Use this as a starting point and adapt it to your needs.

{{< /admonition >}}

At a high level, the BOM for the project includes:

- ESP32-C3 mini module
- Bosch BME680 sensor / Co2 sensor
- QDX50D air pressure sensor
- AC to DC power supply for interfacing with existing 24VAC HVAC power
- Custom PCB

The assembled setup looks like this:

{{<figure name="feature-sense01">}}

Installed in the HVAC system, it appears like this:

{{<figure name="sense02">}}

## ESPHome Config

{{<figure name="ha">}}

Below is a snippet of the ESPHome configuration file for this project. It’s similar to my configuration, but I've removed boilerplate sections (e.g., `mqtt`, `wifi`, etc.) for brevity.

I've included numerous in-line comments to clarify each section. To briefly recap:

- Press the "Clear High Water Mark" button whenever you change the filter to reset the high water mark.
- Ensure that `glbl_air_pressure_delta_clogged_pa` and `glbl_air_pressure_delta_virgin_pa` fields contain values tailored to your system.
- To calibrate these values, install a new filter and note the value reported by `sense_air_pressure_delta_raw_pa`. As the filter clogs over time, this value will rise. When the filter blocks most light when held up to a bright light, it’s time for a replacement. Record the value of `sense_air_pressure_delta_raw_pa` at this point and use it as the "clogged" threshold.

```yaml
##
# The air pressure probe seems to sample continuously; the display is updated a few times per second.
# This is a bit overkill for this application as I really only need to check occasionally while the HVAC fan is running.
# I say occasionally because the filter is not going to clog up in a matter of seconds; it will take many weeks.
# There's little value in knowing that the filter is dropping 2 more Pascals this second compared to what it was 
#   at 10 seconds ago. The value is in knowing that the filter is dropping 2000 more Pascals now than it was four
#   months ago!
#
# Additionally, when measuring using individual Pascals, the readings can be quite noisy!
# Just being near the probe can warm up the ambient ari enough to register a few pascals of pressure change.
# Breathing / talking / just walking past the probe can also cause the readings to jump around a bit!
#
# The point that I'm trying to make here is that we are concerned with the overall trend of the pressure readings, not
#   instantaneous values.
# This isn't quite so straightforward, however since we don't know when the fan is running!
# It may not run for months at a time if the weather is pleasant... or it might be running for days at a time if it's not.
# While it would certainly be the most reliable indicator, electrically interfacing with the 240v blower motor is way, way
#   out of scope for this project!
#
# Fortunately, we have a pretty reliable backup method that allows us to infer when the fan is running with a high degree of
#   confidence: the air pressure probe!
# When the fan is off, the measured air pressure difference will be pretty small since there is (virtually) no flow through the filter.
# Sure, it will bounce around a bit due to environmental factors but the overall measured delta will not be very large.
# When the fan is running, the pressure drop across the filter will be much larger than the idle/ambient value.
#
# If we know what the air pressure drop is across a brand new / virgin filter, we can use that as a threshold; if the measured
#   pressure delta is lower than this threshold, the fan is (probably) not running. If above, the fan is almost certainly running.
#
# Ok, we can infer if the fan is running or not given a measurement, but how do we know when to sample the pressure delta?
# Because the blower motor is going to run for AT LEAST a few hundred seconds at a time, we don't need to sample the
#   pressure delta very often. 2x per minute should be more than enough opportunity to catch the fan running.
#
# Knowing that the pressure delta will only increase as the filter gets clogged, we can compare the current air pressure delta
#   to the absolute maximum value to estimate the remaining life of the filter.
#
# To prevent the estimated filter life percentages from going all over the place, we use a high-water-marker approach.
# This isn't _perfect_ since we'd ideally be keeping track of the average pressure delta but setting that up in ESPHome
#    isn't trivial and the high-water-mark approach is good enough for this application.
##
# So, this means that we need to supply two calibration values for most of the code below to work properly:
#   - The pressure drop across a virgin / brand new filter. This value should be adjusted if the type of filter
#       is changed (e.g. MERV 8 to MERV 13). This value can be obtained "automatically" by just installing a new filter and
#       clicking the "reset high-water mark" button. The next time that the fan runs, the probe will record the highest value
#       and this is a good estimate of the virgin value. Maybe round down just a bit to be safe.
#   - The pressure drop across an end-of-life/clogged filter. The best way to get this value is to use a filter that's
#       about to be replaced. In testing, the difference between a new filter and a clogged filter was about 20-30 hPa
#       so it is possible to estimate this value if you only have the virgin value. If using the estimation approach, check the
#       filter from time to time to see how dirty it is and adjust the clogged value as needed. Once the filter is so dirty that
#       you can't see sunlight through it, it's probably time to replace it. Record the high water marker value at this time
#       and use that as the clogged value. Maybe round down just a bit to be safe.
##
substitutions:

  # Remember, the sensor is configured to report in Pascals but the user is more likely to think in hPa
  # hPa = 100 Pascals
  template_number_input_min_hpa: "10"
  template_number_input_max_hpa: "150"

# We do need all of these values to persist across reboots; for this reason, do not use an ESP8266!
# Internally, this config uses Pascals but the user is more likely to think in hPa so we scale by 100 in each direction
globals:
  # High water marker
  - id: glbl_air_pressure_delta_hwm_pa
    type: float
    restore_value: yes
    initial_value: "0.0"

  # The air pressure delta we'd expect to see for a fresh filter
  - id: glbl_air_pressure_delta_virgin_pa
    type: int
    restore_value: yes
    # Probe is in Pascals and a new filter is about 70 hPa
    initial_value: "70000"

  # And for a clogged filter
  - id: glbl_air_pressure_delta_clogged_pa
    type: int
    restore_value: yes
    # In testing, a very dirty filter has a roughly 30 hPa delta
    initial_value: "95000"

uart:
  - id: uart_co2
    # < ... Omitted ...>

  - id: uart_modbus
    # < ... Omitted ...>

modbus:
  - id: bus_mod_1
    uart_id: uart_modbus

modbus_controller:
  - id: modbus_device
    modbus_id: bus_mod_1
    # Datasheet defaults to addr 01
    address: 0x01
    # See note above about sampling rate; 2x/min should be fine as long as the fan runs for at least 5 min at a time
    update_interval: 30s
    setup_priority: -10


# Note: The air pressure sensor requires a bit of configuration and setup before it can be used with this ESPHome configuration.
# This ESPHome configuration assumes that the sensor has already been configured to 9600 baud, address 0x01
#   and that the reading is in Pascals and has been set to the appropriate level of precision (xx.yy Pascals)
# It might be possible to write a custom script / component / ESPHome automation to write out the necessary configuration from
#   within ESPHome but I didn't see a need to port the python script to ESPHome.
#
# https://github.com/kquinsland/qdx50d-air-pressure-sensor-poc/
##
sensor:
  # This sensor is the instantaneous air pressure value. This sensor will be updated at the rate outlined by the
  # modbus_controller.update_interval value
  # It is expected that teh measured value will be noisy at idle but will stabilize when the HVAC fan is running.
  # Despite the value from this entity being used in other sensors, we do NOT mark this as internal since we want the raw value
  #   to be available in HA for advanced diagnostic / tuning purposes.
  - name: "Air Pressure Delta (Raw)"
    platform: modbus_controller
    id: sense_air_pressure_delta_raw_pa
    modbus_controller_id: modbus_device
    register_type: holding
    address: 0x04
    device_class: atmospheric_pressure
    unit_of_measurement: "Pa"
    state_class: measurement
    icon: "mdi:gauge"
    value_type: S_WORD
    # When we get a new reading update the various presentation layers that depend on us
    on_value:
      # Pass value to the smoothed sensor
      - lambda: id(sense_air_pressure_delta_smoothed_pa).publish_state(id(sense_air_pressure_delta_raw_pa).state);
      # Update the global high water mark, but only if the incoming value is at least as high as the virgin value
      # Until we see a value equal to / higher than virgin, we can assume that the fan is not running.
      - lambda: |-
          // Use absolute value so we don't care if the probe is installed backwards or not; we're only interested in the magnitude
          float abs_x = fabs(x);
          ESP_LOGD("sense_air_pressure_delta_raw_pa.on_value", "[RAW] abs_x: %f", abs_x);

          // If the blower motor is off, the values will be quite small and noisy.
          // We don't bother with any value that's less than the virgin value
          if (abs_x < id(glbl_air_pressure_delta_virgin_pa) ) {
            ESP_LOGD("sense_air_pressure_delta_raw_pa.on_value", "abs_x: %f IS BELOW VIRGIN: %d", abs_x, id(glbl_air_pressure_delta_virgin_pa));
          } else {
            // We are at or above the virgin value so the blower motor is almost certainly running
            ESP_LOGD("sense_air_pressure_delta_raw_pa.on_value", "abs_x: %f IS ABOVE VIRGIN: %d", abs_x, id(glbl_air_pressure_delta_virgin_pa));

            if (abs_x > id(glbl_air_pressure_delta_hwm_pa) ) {
              id(glbl_air_pressure_delta_hwm_pa) = abs_x;
              // Update the hwm sensor (which kicks off other updates)
              id(sense_air_pressure_delta_hwm_pa).update();
            }
          }
          // Log the new values
          ESP_LOGD("sense_air_pressure_delta_raw_pa.on_value", "glbl_air_pressure_delta_hwm_pa: %f", id(glbl_air_pressure_delta_hwm_pa));

  - name: "Air Pressure Delta (Smoothed)"
    platform: template
    id: sense_air_pressure_delta_smoothed_pa
    device_class: atmospheric_pressure
    unit_of_measurement: "Pa"
    state_class: measurement
    icon: "mdi:gauge"
    # We are template, driven by the raw sensor
    update_interval: never
    expire_after: 120s
    filters:
      - exponential_moving_average:
          alpha: .2
          send_every: 15

  # ESPHome has a few different ways of interfacing with the BME680
  # The `bme68x_bsec2` platform requires arduino base for ESPHome and has two large drawbacks:
  #   1. License prevents distributing binary; not a problem here, but annoying for some / in principle
  #   2. Does not permit arbitrary `update_interval` settings. Only choice is 3s or every 300s!
  #
  # It appears that the only real difference between the two platforms is that the proprietary one
  #   has a calculated VOC and CO2 sensor. 
  # This build already has a real CO2 sensor and - while better than nothing - I'd rather have a real VOC sensor as well.
  ##
  - platform: bme680
    # < ... Omitted ...>

  - platform: mhz19
    uart_id: uart_co2
    # < ... Omitted ...>

  # For debugging purposes, show the current value of glbl_air_pressure_delta_hwm_pa
  - name: "Air Pressure Delta High Water Mark"
    platform: template
    id: sense_air_pressure_delta_hwm_pa
    entity_category: diagnostic
    device_class: atmospheric_pressure
    unit_of_measurement: "Pa"
    state_class: TOTAL_INCREASING
    update_interval: never
    lambda: |-
      return id(glbl_air_pressure_delta_hwm_pa);
    on_value:
      - lambda: id(sense_est_filter_life_remaining).update();

  - name: "Estimated Filter Life Remaining"
    platform: template
    id: sense_est_filter_life_remaining
    entity_category: diagnostic
    unit_of_measurement: "%"
    # Technically, this is true but the value could change in a negative direction if the user changes the virgin/clogged values
    # Home Assistant does NOT like it when this class of measurement goes down unless it goes back to zero
    # state_class: TOTAL_INCREASING
    # When the HWM is updated, we'll be called to update
    update_interval: never
    lambda: |-
      if (id(glbl_air_pressure_delta_hwm_pa) < id(glbl_air_pressure_delta_virgin_pa)) {
        ESP_LOGW("sense_est_filter_life_remaining", "[NaN CASE] glbl_air_pressure_delta_hwm_pa: %f glbl_air_pressure_delta_virgin_pa: %f", id(glbl_air_pressure_delta_hwm_pa), id(glbl_air_pressure_delta_virgin_pa));
        return NAN;
      } else {
        ESP_LOGD("sense_est_filter_life_remaining", "[CalcCase] glbl_air_pressure_delta_hwm_pa: %f glbl_air_pressure_delta_virgin_pa: %f, glbl_air_pressure_delta_clogged_pa: %f", id(glbl_air_pressure_delta_hwm_pa), id(glbl_air_pressure_delta_virgin_pa), id(glbl_air_pressure_delta_clogged_pa));
        return 100 - ((id(glbl_air_pressure_delta_hwm_pa) - id(glbl_air_pressure_delta_virgin_pa)) / (id(glbl_air_pressure_delta_clogged_pa) - id(glbl_air_pressure_delta_virgin_pa)) * 100);
      }

# Button to clear the high water mark
# This button should be pressed when a new filter is installed
button:
  - name: "Clear High Water Mark"
    platform: template
    id: btn_clear_hwm
    entity_category: config
    icon: "mdi:refresh"
    on_press:
      - globals.set:
          id: glbl_air_pressure_delta_hwm_pa
          value: "0.0"
      - sensor.template.publish:
          id: sense_air_pressure_delta_hwm_pa
          state: !lambda "return id(glbl_air_pressure_delta_hwm_pa);"
      - sensor.template.publish:
          id: sense_est_filter_life_remaining
          state: !lambda "return NAN;"

# And allow the user to set what a virgin/clogged filter looks like w/o recompiling
# Store the values in Pascals but present them in hPa, use abs() since we only care
#   about the magnitude; not direction.
# Any time we are set, we need to re-calculate the estimated filter life
number:
  - name: "Virgin Filter Pressure Delta"
    platform: template
    id: num_virgin_filter_pressure_delta
    entity_category: config
    icon: "mdi:gauge"
    # Internally, we use Pascals but the user is more likely to think in hPa so we
    # scale by 100 in each direction
    unit_of_measurement: "hPa"
    device_class: atmospheric_pressure
    min_value: ${template_number_input_min_hpa}
    max_value: ${template_number_input_max_hpa}
    step: 1
    lambda: |-
      return id(glbl_air_pressure_delta_virgin_pa)/100;
    set_action:
      - lambda: |-
          id(glbl_air_pressure_delta_virgin_pa) = abs(x)*100;
      - lambda: |-
          id(sense_est_filter_life_remaining).update();

  - name: "Clogged Filter Pressure Delta"
    platform: template
    id: num_virgin_filter_pressure_clogged
    entity_category: config
    icon: "mdi:gauge"
    unit_of_measurement: "hPa"
    device_class: atmospheric_pressure
    min_value: ${template_number_input_min_hpa}
    max_value: ${template_number_input_max_hpa}
    step: 1
    lambda: |-
      return id(glbl_air_pressure_delta_clogged_pa)/100;
    set_action:
      - lambda: |-
          id(glbl_air_pressure_delta_clogged_pa) = abs(x)*100;
      - lambda: |-
          id(sense_est_filter_life_remaining).update();
```

