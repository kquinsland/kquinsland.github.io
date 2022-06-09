# Using ESPHome with the Treatlife DS03


{{< admonition tip "Post depreciation notice" >}}
This is an update to the [`Fixing Home Assistant discovery with Tasmota on the Treatlife DS03` post]({{<relref "posts/2021/05/Treatlife-DS03-Tasmota-Autoconfig-With-Homeassistant" >}}).
{{< /admonition >}}

# Tasmota reliability

A few months ago, I started to notice some bizarre behavior with the DS03 ceiling fan controllers that I had previously flashed with Tasmota.
Very regularly, the devices would crash and reboot! I almost never noticed unless I was explicitly looking at the uptime / boot count graphs for the devices but every once in a while, the device would reboot _right as I was trying to control it remotely_.

I was able to confirm that the rules I was using to 'augment' the Home Assistant auto-discovery payload were the culprit. Odd since nothing had changed; only the version of tasmota has been changing.

While it's not clear _what_ the issue is, it **probably** has something to do with RAM exhaustion.
The details are [in Tasmota issue #15636](https://github.com/arendst/Tasmota/issues/15636#issuecomment-1130511474) if your curious.

The Tasmota rules were a workaround to a broader problem: tasmota offers very little to customize _how_ the entity is advertised to Home Assistant. Since the workaround was no longer working, the next logical step is to re-evaluate if Tasmota is the appropriate firmware for the device.

# Using ESPHome with the DS03

{{< admonition info "Beta software release" >}}
I **strongly** prefer MQTT for my Home Automation interoperability.

The MQTT <-> Home Assistant mechanism in ESPHome has been playing second fiddle to the native ESPHome <-> Home Assistant API as of late so there are more bugs and usability issues. As such, the YAML below depends on a fix for `mqtt.fan` component that exists in the  [2022.06.b2](https://github.com/esphome/esphome/releases/tag/2022.6.0b2) or later release.

Specifically, [this](https://github.com/esphome/esphome/pull/3537) commit.

If you do not use MQTT for your ESPHome <-> Home Assistant linking, you should be fine using the latest 'stable' release of ESPHome.
{{< /admonition >}}

After [migrating the tasmota install to ESPHome](https://esphome.io/guides/migrate_sonoff_tasmota.html), I am happy to report that ESPHome is running on the DS03 and that the full/proper MQTT auto-discovery payload is published. 🥳

Here is a 'reference' YAML file that is similar to the ones I use in 'production'. You will need to add your own MQTT/WiFi... etc configuration.

```yaml
substitutions:
  hostname: YourHostNameHere
  friendly_name: YourDeviceNameHere

esphome:
  name: ${hostname}
  platform: ESP8266
  board: esp01_1m

# Enable logging
logger:
  level: INFO

# Uses a TuYa MCU to drive the power components
# See: https://esphome.io/components/tuya.html
##
tuya:

# See: https://esphome.io/components/uart.html
uart:
  rx_pin: GPIO3
  tx_pin: GPIO1
  # Not sure why/where/how this error shows up but seems functional as is /shrug
  # [E][uart:015]:   Invalid baud_rate: Integration requested baud_rate 9600 but you have 115200!
  baud_rate: 115200

# See: https://esphome.io/components/light/tuya.html
light:
  - name: ${friendly_name} Light
    id: device_light
    platform: "tuya"
    dimmer_datapoint: 10
    switch_datapoint: 9
    min_value: 100
    max_value: 1000
    icon: "mdi:ceiling-fan-light"
    restore_mode: ALWAYS_OFF

# See: https://esphome.io/components/fan/tuya.html
fan:
  - name: ${friendly_name} Speed
    id: device_fan
    platform: "tuya"
    switch_datapoint: 1
    speed_datapoint: 3
    speed_count: 4
    icon: "mdi:ceiling-fan"
    restore_mode: ALWAYS_OFF
```

