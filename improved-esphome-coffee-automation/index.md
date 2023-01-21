# Integrating a dumb coffee maker with Home Assistant via ESPHome

<!-- markdownlint-disable-file MD002 -->

My beloved coffee maker of 10 years has finally died 😢.
Parts are no longer available from either the manufacturer or the second-hand market.

Taking advantage of a (slight) holiday sale discount, I pulled the trigger on a coffee maker that's [designed to be repairable forever](https://us.moccamaster.com/pages/sustainability-at-our-core).
The perpetual serviceability is a side effect of an ultra-simple design; this coffee maker has _zero_ intelligent features which means there's next to no remote control or customizability.

## Adding Home Assistant integration

Fortunately, we can add a decent amount of control with a basic smart outlet.
Naturally, I went with the amazing Sonoff S31 but you can get similar results with _any_ ESPHome compatible device so long as you have a way to control power to the coffee maker and to monitor the power used by the coffee maker.

This borrows a technique that I first wrote about in [`Dynamic timers in ESPHome`]({{< relref "posts/2022/08/esphome-dynamic-timer" >}}) and tweaks it a bit to add of coffee-specific automations:

- Run the boiler for a moment, then pause. This is to let the [coffee bloom](https://www.homegrounds.co/coffee-bloom/).
- Turn relay off automatically after a period of no power use. This is done locally for safety and to save me the hassle of putting a template "is coffee brewing" sensor in home assistant. I still report energy data to home assistant for statistical purposes but don't use the data for automations.

Here's what that looks like:

{{<figure name="cover-ha_device_page">}}

### ESPHome

Below is a simplified ESPHome configuration file that resembles what I use "in production".
As is, you'll need to fill it out / add in your own versions of some of the unique-to-me details intentionally omitted; these will be the device/component `id:` values, mostly.

```yaml
substitutions:
  friendly_name: "Mr. Coffee"
  friendly_name_short: "Coffee"

esphome:
  name: ${hostname}

esp8266:
  board: esp01_1m

  restore_from_flash: false

packages:
  # <...>
  # Has ID: out_relay_1
  output: !include ../../packages/sonoff_s31_outlet/output_relay.yaml

globals:
  # Does the button toggle the relay
  - id: glbl_relay_latched
    type: bool
    restore_value: no
    initial_value: "true"

  ##
  # Bloom settings
  ##
  # How long do we run the heater for?
  # The time from power on to first water in basket depends on water temperature.
  # The cooler the water, the longer it'll be.
  # For now, the intent is to use other sensors to estimate the water temp and have HA
  #   update the configuration before starting a brew.
  #
  # Maybe in a future version, outfit with water level/temp probe directly.
  ##
  - id: glbl_pre_bloom_time_sec
    type: int
    restore_value: yes
    # Note: value for demonstration purposes; do your own testing to determine appropriate value for your
    #   environment / coffee maker
    initial_value: "30"

  # How long do we pause to do a bloom for?
  - id: glbl_bloom_time_sec
    type: int
    restore_value: yes
    # Note: value for demonstration purposes; do your own testing to determine appropriate value for your
    #   environment / coffee maker
    initial_value: "30"

  # And of course we need something to hold the timer value
  - id: _glbl_brew_timer_ticks
    type: int
    restore_value: no
    initial_value: "0"

  # We need a place to store the number of ticks that we have observed low power while brewing.
  - id: _glbl_brew_low_power_ticks
    type: int
    restore_value: no
    initial_value: "0"

##
# See: https://esphome.io/components/sensor/index.html
sensor:
  - platform: cse7766
    update_interval: 1s
    # <...>
    power:
      name: "${friendly_name_short} Power"
      accuracy_decimals: 1
      # Needed for total daily calculations
      id: s31_power
      filters:
        - or:
            - throttle: 300s
            # Publish every time there's been more than 3.5W change
            - delta: 3.5

  # Do the integration locally so HA does not have to
  # See: https://esphome.io/components/sensor/total_daily_energy.html
  ##
  - name: "${friendly_name_short} Daily Energy"
    platform: total_daily_energy
    power_id: s31_power
    icon: "mdi:meter-electric-outline"

# On the device page, HA will display a widget so we can adjust bloom time
number:
  - name: "Pre Bloom Time"
    id: num_pre_bloom_sec
    platform: template
    icon: "mdi:timer-sand"
    entity_category: "config"
    unit_of_measurement: seconds
    mode: slide
    # Note: value for demonstration purposes; do your own testing to determine appropriate value for your
    #   environment / coffee maker
    min_value: 10
    max_value: 60
    step: 2

    lambda: |-
      return (int) id(glbl_pre_bloom_time_sec);

    set_action:
      then:
        - globals.set:
            id: glbl_pre_bloom_time_sec
            value: !lambda |-
              // TODO: we're relying on HA to pass an integer; perhaps we should do atoi() and catch any exceptions
              return (int) x;

  - name: "Bloom Time"
    id: num_bloom_sec
    platform: template
    icon: "mdi:timer-sand-paused"
    entity_category: "config"
    unit_of_measurement: seconds
    mode: slider
    # Note: value for demonstration purposes; do your own testing to determine appropriate value for your
    #   environment / coffee maker
    min_value: 10
    max_value: 60
    step: 2

    lambda: |-
      return (int) id(glbl_bloom_time_sec);

    set_action:
      then:
        - globals.set:
            id: glbl_bloom_time_sec
            value: !lambda |-
              // TODO: we're relying on HA to pass an integer; perhaps we should do atoi() and catch any exceptions
              return (int) x;

# Show current phase (IDLE/BREW/BLOOM/NEEDS-CLEANING...etc)
text_sensor:
  - name: "Mode"
    id: txt_operation_mode
    platform: template
    icon: "mdi:information-off-outline"
    entity_category: "diagnostic"
    # Will be called to update as necessary from other components
    update_interval: never
    lambda: |
      // This lambda function should never be called / we should never update the text sensor this way
      return {"Unknown"};

# Not sure if useful but including it anyways.
switch:
  - name: "${friendly_name_short} Relay Latch"
    platform: template
    id: sw_relay_mode
    device_class: "switch"
    icon: "mdi:link-variant"
    entity_category: "config"

    lambda: |-
      if (id(glbl_relay_latched)) {
        return true;
      } else {
        return false;
      }

    turn_on_action:
      - globals.set:
          id: glbl_relay_latched
          value: "true"

    turn_off_action:
      - globals.set:
          id: glbl_relay_latched
          value: "false"

  - name: "${friendly_name_short} Power"
    id: sw_relay_toggle
    platform: output
    output: out_relay_1
    icon: "mdi:coffee-maker-outline"
    restore_mode: ALWAYS_OFF
    device_class: "switch"

    # As soon as the power is turned on, start the timer
    on_turn_on:
      - script.execute: _brew_timer_tick

    # And if turned off, reset
    on_turn_off:
      - script.execute: _on_brew_turned_off

binary_sensor:
  # Make the manual activation button behave the same as "stock".
  # May use for some future functionality:
  #   - Triple click to indicate that I want to descale / ignore bloom settings
  #   - Double click to indicate that the water / grounds have been refreshed
  ##
  - name: "${friendly_name_short} Button"
    platform: gpio
    internal: true
    pin:
      number: GPIO0
      mode: INPUT_PULLUP
      inverted: True

    # Press is momentary quick
    on_press:
      then:
        - if:
            condition:
              # If we are in latched mode
              lambda: 'return id(glbl_relay_latched);'
            then:
              - switch.toggle:
                  id: sw_relay_toggle

            else:
              - logger.log:
                  level: DEBUG
                  format: "Button1 pressed but relays unlinked"

# Every second we need to increment the timer
script:
  - id: _brew_timer_tick
    # Start a new run after previous runs completes. This will happen until timer.stop() is called on us
    ##
    mode: queued
    max_runs: 0
    then:
      # A single 'tick' is 1 second long
      - delay: 1s
      - lambda: |-
          /*
            Called every second while relay is on.
            Compare number of invocations to user settings to figure out which phase we should be in.
            Update the text sensor and turn switch on/off as needed.
          */
          auto const static TAG = "lambda._brew_timer_tick";
          // Number of seconds post bloom with low power before we transition from brewing to idle.
          static int idle_ticks_threshold = 30;
          static int num_ticks = 0;
          static int total_bloom_time_sec = 0;
          static bool post_bloom = false;
          total_bloom_time_sec = (id(glbl_bloom_time_sec) + id(glbl_pre_bloom_time_sec));
          ESP_LOGD(TAG, "Idle. num_ticks: %i, total_bloom_time_sec: %i", num_ticks, total_bloom_time_sec);

          // Count execution
          id(_glbl_brew_timer_ticks) += 1;
          num_ticks = id(_glbl_brew_timer_ticks);

          // If the switch has been turned off - for any reason - we cancel any scheduled executions and don't continue
          if( !id(sw_relay_toggle) ) {
            id(_brew_timer_tick).stop();
            return;
          }

          // If the brew timer ticks counter is 0, assume IDLE
          if (num_ticks == 0) {
            ESP_LOGE(TAG, "Idle. _glbl_brew_timer_ticks: %i", _glbl_brew_timer_ticks);
            id(txt_operation_mode).publish_state("Idle");
          }

          // If we are between 1 and glbl_pre_bloom_time_sec then brewing -> Blooming
          if( (num_ticks >= 1 ) && (num_ticks <= id(glbl_pre_bloom_time_sec)) ) {
            id(txt_operation_mode).publish_state("Pre Bloom");
          }

          // If we are between glbl_pre_bloom_time_sec and glbl_bloom_time_sec then blooming.
          if( (num_ticks > id(glbl_pre_bloom_time_sec) ) && (num_ticks <= total_bloom_time_sec ) ) {
            id(txt_operation_mode).publish_state("Blooming");
            // Note, we turn the relay off directly and not the switch. We want the switch / web UI to still show "on/brewing".
            // The text sensor will show that we are blooming
            id(out_relay_1).turn_off();
          }

          // Otherwise we are brewing.
          if(num_ticks > total_bloom_time_sec ) {
            id(txt_operation_mode).publish_state("Brewing");
            id(out_relay_1).turn_on();
            post_bloom = true;
          }

          // After bloom, start counting the number of times we observe low power.
          // After T seconds of no power use, we assume that we're out of water and reset everything.
          ESP_LOGD(TAG, "post bloom: %i, Current power consumption: %f Watts", post_bloom, id(s31_power).state);
          if ( post_bloom && (id(s31_power).state < 2)) {
            id(_glbl_brew_low_power_ticks) +=1;
            ESP_LOGD(TAG, "_glbl_brew_low_power_ticks: %i", id(_glbl_brew_low_power_ticks));
          }

          if (id(_glbl_brew_low_power_ticks) >= idle_ticks_threshold){
            ESP_LOGD(TAG, "Assuming done with brew as _glbl_brew_low_power_ticks: %i >= %i", id(_glbl_brew_low_power_ticks), idle_ticks_threshold);
            id(sw_relay_toggle).turn_off();
            return;
          }

          //re-schedule so we're called again in a second!
          id(_brew_timer_tick).execute();


  - id: _on_brew_turned_off
    mode: single
    then:
    # Disable timer, have the text sensor update
    - lambda: |-
        id(_glbl_brew_timer_ticks) = 0;
        id(_glbl_brew_low_power_ticks) = 0;
        id(txt_operation_mode).publish_state("Idle");
        id(_brew_timer_tick).stop();
```

### Home Assistant automations

When some automation determines that it's now time to start making coffee, this is a portion of the script that is executed:

```yaml

sequence:
  # ...
- service: number.set_value
    data:
      # This is a bit simplified; my template does some more maths to map the temperature to a more precise value.
      # I simplified the value template here to illustrate how the adjustable timers are meant to be used.
      value: >-
        {% if states('sensor.kitchen_temperature')|float(-1) < 25 %}35{% else
        %}20{% endif %}
    target:
      entity_id: number.pre_bloom_time
  - service: number.set_value
    data:
      value: "35.0"
    target:
      entity_id: number.bloom_time
  - type: turn_on
    device_id: #< Your Device ID Here >
    entity_id: switch.coffee_power
    domain: switch
```

