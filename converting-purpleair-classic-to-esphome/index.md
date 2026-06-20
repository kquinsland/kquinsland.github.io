# Converting PurpleAir Classic to ESPHome

LLM assisted firmware reverse engineering and ESPHome conversion of a PurpleAir Classic air quality sensor.

Canonical: https://karlquinsland.com/converting-purpleair-classic-to-esphome/
Published: 2026-06-20
Updated: 2026-06-20
Tags: esphome, mqtt, home-assistant, two-minute-teardown


---

# Converting a PurpleAir Classic to ESPHome

I still don't 100% know what the issue is but for the past several months, my [PurpleAir Classic](https://www2.purpleair.com/products/purpleair-pa-ii) sensor has been incredibly flakey when it comes to reporting data.
Home Assistant would report random bouts of "timeouts" while trying to poll the sensor and that would result in gaps in the data.
In the best case, the gaps were only a few minutes long but sometimes they would last for _hours_ at a time.

I rely on the readings from this unit to inform other automations around when it's a good time to open windows versus use the AC so I wanted to find a more reliable solution.

Somewhere in the middle of frantically troubleshooting all of this, I realized/remembered that this is just an ESP8266 based device so it _should_ be straight-forward to convert it to run ESPHome instead of the stock firmware.

![images_render](https://karlquinsland.com/converting-purpleair-classic-to-esphome/images/render.webp)


Turns out, yeah, it was pretty straight forward.

## But why?

To their credit, the PurpleAir team does not make it _hard_ to get the data off of your device directly.
They have a [local API](https://community.purpleair.com/t/local-json-endpoint-documentation/6097) that you can query for the raw sensor data.

Combine this with a bunch of [RESTful sensors in Home Assistant](https://www.home-assistant.io/integrations/sensor.rest/) and you _can_ get the data into your home automation system without too much trouble.

You will still need a firewall rule to prevent the device from phoning home to the PurpleAir cloud if desired.

Converting to ESPHome, however, has a few advantages:

- I get full control over the firmware and the transport(s) used to get data off the device and into Home Assistant. As noted above, _something_ about the stock firmware was flakey and resulted in data gaps. ESPhome lets me move to rock-solid reliable MQTT.

- The stock firmware does not support IPv6 at all. ESPHome still [only has _ok-ish_ support for IPv6](https://www.youtube.com/watch?v=5TxFjNoNoHY) but "some" is better than "none" and I will _never_ achieve my goal of moving to a fully IPv6 home network if I have devices that don't support it.

- Much simpler Home Assistant configuration. With the stock firmware, I had to set up a bunch of RESTful sensors and then use a ton of template jinja to extract the useful data from the `/json` endpoint on the device. A side effect of this is that HomeAssistant never had a "device" to own the various entities. This is no longer the case with ESPHome. I define everything once in YAML and as soon as the device comes online, Home Assistant automatically discovers it and all of the sensors that I defined in the ESPHome config and creates entities for them under a single device.

## Teardown

I couldn't find any authoritative schematics or teardowns online so I didn't know what pins on the ESP module would be wired up to which sensors.

One reliable way to answer those questions is with a teardown and manual reverse-engineering of the PCB.
But I wanted to try another approach first; could I dump the firmware and set up a little container for Codex with `--yolo` enabled and see if it could figure out the pinout information for me?
And could it do it faster than I could?

Let's find out :).

There is only one philips screw on the exterior ot the device.
Remove it and then the grey PVC 'ring' that holds the components can be gently pried off.

Once you get the ring off, the components will slid out of the cap:

![images_td_02](https://karlquinsland.com/converting-purpleair-classic-to-esphome/images/td_02.webp)


And here's the same thing from a different angle:

![images_td_01](https://karlquinsland.com/converting-purpleair-classic-to-esphome/images/td_01.webp)


I didn't take the teardown any further because it's all covered in Kapton tape that I can't really replace and I don't want to risk damaging anything.
And it was at about this time that Codex notified me that it had some strong guesses about the pinout for the BME280 and PMS5003 sensors so I figured that I didn't really _need_ to dig deeper unless those guesses turned out to be off base.

As far as I can tell, this is a _super_ early version of the hardware; the grey PVC ring that holds the components was clearly modified by hand to accommodate the sensors.
The Kapton tape is all hand-applied as well.

To be clear, nothing wrong with that!
It's just interesting to see that they keep the same PVC cap enclosure on [newer models](https://www2.purpleair.com/products/list) but it appears that the internals now have a custom plastic frame to position components in a manner that is a bit more polished than what I have.

### So who won?

It was a tie... sorta.
Using nothing more than basic ad-hock python tools (<3 `uv`), Codex took about 20 min to figure out:

- They used a pretty standard `app0` and `app1` partition scheme for the firmware
- The applications on each partition were different versions; Codex took both apart and compared what changed between the older version and the newer version
- Figured out the formula/constants used to calculate the AQI metric from the raw sensor data
- Determined that the firmware supports some optional (development?) hardware:
  - a RTC modules
  - logging to an SD card

In any case, Codex was very confident that it knew the i2c pins for the BME280 but it was less confident about the UART pins for the PMS5003 sensors and which - if any GPIO - were used to enable/disable the sensors.

I asked it to write a basic ESPHome config that would compile down to a firmware that looped over the possible pin combinations for the PMS5003 sensors and to report that back over the USB/UART connection.
It did so and after several minutes of watching Codex try different pin combinations, it was able to figure out how the module talks to both the BME280 and the PMS5003 sensors.

## Codex Notes

In any case, here are a few snips from the Codex conversation/notepad.
Everything below is more or less verbatim from Codex.
I have only made _slight_ edits for formatting and clarity.

### Firmware contents

The 2 MB flash dump contains multiple ESP8266 images:

| Offset     | Contents                      |
| ---------- | ----------------------------- |
| `0x000000` | bootloader                    |
| `0x001000` | app image, `PurpleAir/7.02`   |
| `0x076000` | bootloader                    |
| `0x077000` | app image, `PurpleAir/6.07`   |
| `0x101000` | small SDK/RF/test style image |
| `0x16a000` | bootloader                    |
| `0x16b000` | app image, `PurpleAir/7.02`   |

The v7.02 app was built against the Arduino ESP8266 core 2.5.2.
It contains local web/JSON output, OTA update support, and strings for PurpleAir cloud, ThingSpeak, Google geolocation, NTP, and Azure IoT endpoints.

### Hardware map from the firmware

| Device                         | Interface                               | Pins / address                                                                       | Confidence | Notes                                                                                        |
| ------------------------------ | --------------------------------------- | ------------------------------------------------------------------------------------ | ---------- | -------------------------------------------------------------------------------------------- |
| `BME280` / `BME680`            | I2C                                     | SDA `GPIO4`, SCL `GPIO5`; firmware tries `0x77`, then `0x76`; live scan found `0x76` | high       | `BSEC` strings are present for BME680 IAQ support.                                           |
| PMSX003 A                      | Plantower UART RX + enable              | RX `GPIO2`, enable `GPIO0` active high                                               | high       | Confirmed with ESPHome probe on this unit.                                                   |
| PMSX003 B                      | Plantower UART RX + enable              | RX `GPIO13`, enable `GPIO12` active high                                             | high       | Confirmed with ESPHome probe on this unit.                                                   |
| PMS firmware-probed alternates | UART-ish input / output-control layouts | `GPIO12`, `GPIO14`                                                                   | medium     | Present in the firmware, but live probes did not need them for the two physical PMS sensors. |
| ADC0                           | ESP8266 ADC                             | ADC0                                                                                 | high       | Exposed as `ADC0` / `adc`.                                                                   |
| `RV3028`, `DS3231`, `KX122`    | likely I2C                              | likely shared `GPIO4` / `GPIO5`                                                      | medium     | Optional support strings; may not be populated.                                              |
| `OPENLOG`                      | serial logging                          | not fully pinned down                                                                | medium-low | Firmware contains `OPENLOG`, `!Baud`, `115200`, `.log`, `.csv`, and `disk` strings.          |

The PMS constructor stores Plantower frame magic bytes `0x42 0x4d` (`B`, `M`) and probes these layouts:

| Channel     | Input pin | Output/control pin | Label |
| ----------- | --------: | -----------------: | ----- |
| A           |  `GPIO12` |            `GPIO0` | `A`   |
| B           |  `GPIO13` |            `GPIO2` | `B`   |
| B alternate |  `GPIO13` |           `GPIO12` | `B`   |
| A alternate |   `GPIO2` |            `GPIO0` | `A`   |
| O / other   |  `GPIO14` |            `GPIO0` | `O`   |

Current best ESPHome wiring model for this unit:

| Function                   |      Pin | State                                                |
| -------------------------- | -------: | ---------------------------------------------------- |
| PMS A UART RX              |  `GPIO2` | 9600 baud                                            |
| PMS A enable / wake / gate |  `GPIO0` | high = PMS A frames, low = no sustained PMS A frames |
| PMS B UART RX              | `GPIO13` | 9600 baud                                            |
| PMS B enable / wake / gate | `GPIO12` | high = PMS B frames, low = no sustained PMS B frames |
| BME I2C SDA                |  `GPIO4` | I2C                                                  |
| BME I2C SCL                |  `GPIO5` | I2C                                                  |

To switch between the two PMS sensors, assert one enable pin and de-assert the other:

| Selected sensor | `GPIO0` | `GPIO12` | Read from |
| --------------- | ------: | -------: | --------- |
| PMS A           |    high |      low | `GPIO2`   |
| PMS B           |     low |     high | `GPIO13`  |

Both sensors appear to have separate UART RX lines into the ESP, so switching is about sensor enable/wake/gate rather than changing one shared UART pin.
In ESPHome, it is still safer to read one at a time on ESP8266 because both PMS inputs use software serial.

### Sensor data and AQI

The firmware publishes raw Plantower PMSX003 fields:

- `pm1_0_cf_1`
- `pm2_5_cf_1`
- `pm10_0_cf_1`
- `pm1_0_atm`
- `pm2_5_atm`
- `pm10_0_atm`
- `p_0_3_um`
- `p_0_5_um`
- `p_1_0_um`
- `p_2_5_um`
- `p_5_0_um`
- `p_10_0_um`

It also computes PM2.5 AQI locally for both `cf_1` and `atm`:

- `pm2.5_aqi_cf_1`
- `pm2.5_aqi_atm`

In the Plantower/PMS field names, `cf_1` means the sensor's `CF=1` standard/factory-calibrated concentration channel.
The `atm` fields are the atmospheric-environment concentration values normally used for ambient air readings.

The AQI function is a standard linear interpolation over breakpoint ranges:

```text
AQI = round(((I_high - I_low) / (C_high - C_low)) * (C - C_low) + I_low)
```

PM2.5 concentration breakpoints found in the firmware:

```text
0, 12, 35, 55, 150, 250, 350, 500
```

AQI breakpoints found in the firmware:

```text
0, 50, 51, 100, 101, 150, 151, 200, 201, 300, 301, 400, 401, 500
```

The firmware uses integer-ish PM2.5 concentration cutoffs (`35`, `55`, etc.) rather than preserving the usual EPA decimal cutoffs (`35.4`, `55.4`, etc.) in the literal table.

## Code

And with that, here's a basic ESPHome configuration that you should be able to use on your own PurpleAir Classic.
This config offers up (almost) all of the same data that was available from the `/json` endpoint on the stock firmware and even does the combining/averaging on the device so you don't have to do it in Home Assistant with a bunch of template sensors like you would with the stock firmware and the RESTFul sensor pattern.

Total time from "wait, I can probably not bother debugging this flaky data availability issue if I just put ESPHome on it" to "have a working ESPHome config that I can flash" was about an hour!

```yaml
substitutions:
    name: purpleair

esphome:
    name: ${name}
    on_boot:
        priority: 600
        then:
            - switch.turn_on: pms_a_enable_gpio0
            - switch.turn_on: pms_b_enable_gpio12
            - delay: 1s
            - lambda: |-
                  id(active_pms_sensor) = 0;
            - script.execute: select_pms_a

esp8266:
    board: esp_wroom_02
    board_flash_mode: dio

logger:
    level: DEBUG
    baud_rate: 115200

i2c:
    sda: GPIO4
    scl: GPIO5
    scan: true

uart:
    - id: pms_a_uart
      rx_pin: GPIO2
      baud_rate: 9600
      rx_buffer_size: 512

    - id: pms_b_uart
      rx_pin: GPIO13
      baud_rate: 9600
      rx_buffer_size: 512

globals:
    - id: active_pms_sensor
      type: int
      restore_value: no
      initial_value: "0"

switch:
    - name: "PMS A Enable GPIO0"
      id: pms_a_enable_gpio0
      platform: gpio
      pin: GPIO0
      restore_mode: ALWAYS_ON
      internal: true

    - name: "PMS B Enable GPIO12"
      id: pms_b_enable_gpio12
      platform: gpio
      pin: GPIO12
      restore_mode: ALWAYS_ON
      internal: true

text_sensor:
    - name: "Active PMS Sensor"
      id: active_pms_sensor_name
      platform: template
      entity_category: diagnostic
      update_interval: 5s
      lambda: |-
          if (id(active_pms_sensor) == 0)
            return std::string("PMS A");
          if (id(active_pms_sensor) == 1)
            return std::string("PMS B");
          return std::string("Unknown");

    - name: "AQI Description"
      id: pms_pm25_aqi_description
      platform: template
      icon: mdi:air-filter
      update_interval: 30s
      lambda: |-
          const float aqi = id(pms_pm25_aqi_atm_average).state;
          if (isnan(aqi))
            return std::string("undefined");
          if (aqi >= 401.0f)
            return std::string("Very Hazardous");
          if (aqi >= 301.0f)
            return std::string("Hazardous");
          if (aqi >= 201.0f)
            return std::string("Very Unhealthy");
          if (aqi >= 151.0f)
            return std::string("Unhealthy");
          if (aqi >= 101.0f)
            return std::string("Unhealthy for Sensitive Groups");
          if (aqi >= 51.0f)
            return std::string("Moderate");
          if (aqi >= 0.0f)
            return std::string("Good");
          return std::string("undefined");

script:
    - id: select_pms_a
      mode: restart
      then:
          - switch.turn_off: pms_b_enable_gpio12
          - delay: 250ms
          - switch.turn_on: pms_a_enable_gpio0
          - lambda: |-
                id(active_pms_sensor) = 0;
                ESP_LOGW("pms_select", "Selected PMS A: RX GPIO2, enable GPIO0=HIGH, GPIO12=LOW");
          - text_sensor.template.publish:
                id: active_pms_sensor_name
                state: "PMS A"

    - id: select_pms_b
      mode: restart
      then:
          - switch.turn_off: pms_a_enable_gpio0
          - delay: 250ms
          - switch.turn_on: pms_b_enable_gpio12
          - lambda: |-
                id(active_pms_sensor) = 1;
                ESP_LOGW("pms_select", "Selected PMS B: RX GPIO13, enable GPIO12=HIGH, GPIO0=LOW");
          - text_sensor.template.publish:
                id: active_pms_sensor_name
                state: "PMS B"

interval:
    - interval: 60s
      # Without a startup delay, ESPHome runs this interval at boot and flips
      # from PMS A to PMS B before A gets its first full sampling window.
      startup_delay: 60s
      then:
          - if:
                condition:
                    lambda: "return id(active_pms_sensor) == 0;"
                then:
                    - script.execute: select_pms_b
                else:
                    - script.execute: select_pms_a

sensor:
    - name: "Uptime"
      platform: uptime
      type: seconds

    - platform: bme280_i2c
      address: 0x76
      temperature:
          name: "BME280 Temperature Raw"
          id: bme_temperature_c
          entity_category: diagnostic
      pressure:
          name: "Pressure"
      humidity:
          name: "BME280 Humidity Raw"
          id: bme_humidity
          entity_category: diagnostic

    - platform: pmsx003
      type: PMSX003
      uart_id: pms_a_uart
      update_interval: 0s
      pm_1_0_std:
          name: "PMS A PM1.0 CF1"
          id: pms_a_pm1_cf1
          internal: true
      pm_2_5_std:
          name: "PMS A PM2.5 CF1"
          id: pms_a_pm25_cf1
          internal: true
      pm_10_0_std:
          name: "PMS A PM10 CF1"
          id: pms_a_pm10_cf1
          internal: true
      pm_1_0:
          name: "PMS A PM1.0 Atmospheric"
          id: pms_a_pm1_atm
          internal: true
      pm_2_5:
          name: "PMS A PM2.5 Atmospheric"
          id: pms_a_pm25_atm
          internal: true
      pm_10_0:
          name: "PMS A PM10 Atmospheric"
          id: pms_a_pm10_atm
          internal: true
      pm_0_3um:
          name: "PMS A Particles 0.3um"
          id: pms_a_particles_0_3um
          internal: true
      pm_0_5um:
          name: "PMS A Particles 0.5um"
          id: pms_a_particles_0_5um
          internal: true
      pm_1_0um:
          name: "PMS A Particles 1.0um"
          id: pms_a_particles_1_0um
          internal: true
      pm_2_5um:
          name: "PMS A Particles 2.5um"
          id: pms_a_particles_2_5um
          internal: true
      pm_5_0um:
          name: "PMS A Particles 5.0um"
          id: pms_a_particles_5_0um
          internal: true
      pm_10_0um:
          name: "PMS A Particles 10.0um"
          id: pms_a_particles_10_0um
          internal: true

    - platform: pmsx003
      type: PMSX003
      uart_id: pms_b_uart
      update_interval: 0s
      pm_1_0_std:
          name: "PMS B PM1.0 CF1"
          id: pms_b_pm1_cf1
          internal: true
      pm_2_5_std:
          name: "PMS B PM2.5 CF1"
          id: pms_b_pm25_cf1
          internal: true
      pm_10_0_std:
          name: "PMS B PM10 CF1"
          id: pms_b_pm10_cf1
          internal: true
      pm_1_0:
          name: "PMS B PM1.0 Atmospheric"
          id: pms_b_pm1_atm
          internal: true
      pm_2_5:
          name: "PMS B PM2.5 Atmospheric"
          id: pms_b_pm25_atm
          internal: true
      pm_10_0:
          name: "PMS B PM10 Atmospheric"
          id: pms_b_pm10_atm
          internal: true
      pm_0_3um:
          name: "PMS B Particles 0.3um"
          id: pms_b_particles_0_3um
          internal: true
      pm_0_5um:
          name: "PMS B Particles 0.5um"
          id: pms_b_particles_0_5um
          internal: true
      pm_1_0um:
          name: "PMS B Particles 1.0um"
          id: pms_b_particles_1_0um
          internal: true
      pm_2_5um:
          name: "PMS B Particles 2.5um"
          id: pms_b_particles_2_5um
          internal: true
      pm_5_0um:
          name: "PMS B Particles 5.0um"
          id: pms_b_particles_5_0um
          internal: true
      pm_10_0um:
          name: "PMS B Particles 10.0um"
          id: pms_b_particles_10_0um
          internal: true

    - name: "Temperature"
      id: purpleair_temperature_c
      platform: template
      unit_of_measurement: "°C"
      device_class: temperature
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 60s
      lambda: |-
          const float c = id(bme_temperature_c).state;
          if (isnan(c))
            return NAN;
          // Match the old HA package/PurpleAir convention: the BME280 reads
          // about 8 F / 4.4 C high in this enclosure.
          return c - (8.0f * 5.0f / 9.0f);

    - name: "Humidity"
      id: purpleair_humidity_corrected
      platform: template
      unit_of_measurement: "%"
      device_class: humidity
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 60s
      lambda: |-
          const float humidity = id(bme_humidity).state;
          if (isnan(humidity))
            return NAN;
          // Match the old HA package/PurpleAir convention: RH is corrected
          // down by 4 percentage points for this enclosure.
          const float corrected = humidity - 4.0f;
          if (corrected < 0.0f)
            return 0.0f;
          if (corrected > 100.0f)
            return 100.0f;
          return corrected;

    - name: "Dewpoint"
      id: purpleair_dewpoint_c
      platform: template
      unit_of_measurement: "°C"
      device_class: temperature
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 60s
      lambda: |-
          const float temp_c = id(purpleair_temperature_c).state;
          const float humidity = id(purpleair_humidity_corrected).state;
          if (isnan(temp_c) || isnan(humidity) || humidity <= 0.0f)
            return NAN;

          const float a = 17.625f;
          const float b = 243.04f;
          const float gamma = logf(humidity / 100.0f) + ((a * temp_c) / (b + temp_c));
          const float dewpoint_c = (b * gamma) / (a - gamma);
          return dewpoint_c;

    - name: "PM1.0 CF1 Average"
      id: pms_pm1_cf1_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm1
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm1_cf1).state;
          const float b = id(pms_b_pm1_cf1).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "PM2.5 CF1 Average"
      id: pms_pm25_cf1_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm25
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm25_cf1).state;
          const float b = id(pms_b_pm25_cf1).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "PM10 CF1 Average"
      id: pms_pm10_cf1_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm10
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm10_cf1).state;
          const float b = id(pms_b_pm10_cf1).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "PM1.0 Atmospheric Average"
      id: pms_pm1_atm_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm1
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm1_atm).state;
          const float b = id(pms_b_pm1_atm).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "PM2.5 Atmospheric Average"
      id: pms_pm25_atm_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm25
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm25_atm).state;
          const float b = id(pms_b_pm25_atm).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "PM10 Atmospheric Average"
      id: pms_pm10_atm_average
      platform: template
      unit_of_measurement: "μg/m³"
      device_class: pm10
      accuracy_decimals: 1
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm10_atm).state;
          const float b = id(pms_b_pm10_atm).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return (a + b) / 2.0f;
          if (a_ok)
            return a;
          if (b_ok)
            return b;
          return NAN;

    - name: "Particles 0.3um Average"
      id: pms_particles_0_3um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_0_3um).state;
          const float b = id(pms_b_particles_0_3um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "Particles 0.5um Average"
      id: pms_particles_0_5um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_0_5um).state;
          const float b = id(pms_b_particles_0_5um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "Particles 1.0um Average"
      id: pms_particles_1_0um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_1_0um).state;
          const float b = id(pms_b_particles_1_0um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "Particles 2.5um Average"
      id: pms_particles_2_5um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_2_5um).state;
          const float b = id(pms_b_particles_2_5um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "Particles 5.0um Average"
      id: pms_particles_5_0um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_5_0um).state;
          const float b = id(pms_b_particles_5_0um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "Particles 10.0um Average"
      id: pms_particles_10_0um_average
      platform: template
      unit_of_measurement: "p/m³"
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_particles_10_0um).state;
          const float b = id(pms_b_particles_10_0um).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);
          if (a_ok && b_ok)
            return ((a + b) / 2.0f) * 10000.0f;
          if (a_ok)
            return a * 10000.0f;
          if (b_ok)
            return b * 10000.0f;
          return NAN;

    - name: "AirQuality A"
      id: pms_a_pm25_aqi_atm
      platform: template
      device_class: aqi
      icon: mdi:air-filter
      accuracy_decimals: 0
      state_class: measurement
      internal: true
      update_interval: 30s
      lambda: |-
          const float c = id(pms_a_pm25_atm).state;
          if (isnan(c) || c < 0.0f)
            return NAN;

          auto aqi = [](float value, float c_low, float c_high, float i_low, float i_high) -> float {
            return roundf(((i_high - i_low) / (c_high - c_low)) * (value - c_low) + i_low);
          };

          if (c <= 12.0f)
            return aqi(c, 0.0f, 12.0f, 0.0f, 50.0f);
          if (c <= 35.0f)
            return aqi(c, 12.0f, 35.0f, 51.0f, 100.0f);
          if (c <= 55.0f)
            return aqi(c, 35.0f, 55.0f, 101.0f, 150.0f);
          if (c <= 150.0f)
            return aqi(c, 55.0f, 150.0f, 151.0f, 200.0f);
          if (c <= 250.0f)
            return aqi(c, 150.0f, 250.0f, 201.0f, 300.0f);
          if (c <= 350.0f)
            return aqi(c, 250.0f, 350.0f, 301.0f, 400.0f);
          if (c <= 500.0f)
            return aqi(c, 350.0f, 500.0f, 401.0f, 500.0f);
          return 500.0f;

    - name: "AirQuality B"
      id: pms_b_pm25_aqi_atm
      platform: template
      device_class: aqi
      icon: mdi:air-filter
      accuracy_decimals: 0
      state_class: measurement
      internal: true
      update_interval: 30s
      lambda: |-
          const float c = id(pms_b_pm25_atm).state;
          if (isnan(c) || c < 0.0f)
            return NAN;

          auto aqi = [](float value, float c_low, float c_high, float i_low, float i_high) -> float {
            return roundf(((i_high - i_low) / (c_high - c_low)) * (value - c_low) + i_low);
          };

          if (c <= 12.0f)
            return aqi(c, 0.0f, 12.0f, 0.0f, 50.0f);
          if (c <= 35.0f)
            return aqi(c, 12.0f, 35.0f, 51.0f, 100.0f);
          if (c <= 55.0f)
            return aqi(c, 35.0f, 55.0f, 101.0f, 150.0f);
          if (c <= 150.0f)
            return aqi(c, 55.0f, 150.0f, 151.0f, 200.0f);
          if (c <= 250.0f)
            return aqi(c, 150.0f, 250.0f, 201.0f, 300.0f);
          if (c <= 350.0f)
            return aqi(c, 250.0f, 350.0f, 301.0f, 400.0f);
          if (c <= 500.0f)
            return aqi(c, 350.0f, 500.0f, 401.0f, 500.0f);
          return 500.0f;

    - name: "AirQuality Combined"
      id: pms_pm25_aqi_atm_average
      platform: template
      device_class: aqi
      icon: mdi:air-filter
      accuracy_decimals: 0
      state_class: measurement
      update_interval: 30s
      lambda: |-
          const float a = id(pms_a_pm25_aqi_atm).state;
          const float b = id(pms_b_pm25_aqi_atm).state;
          const bool a_ok = !isnan(a);
          const bool b_ok = !isnan(b);

          auto round_like_ha_min_max = [](float value) -> float {
            return roundf(value / 10.0f) * 10.0f;
          };

          // ha.yaml used Home Assistant min_max type=mean with round_digits=-1.
          if (a_ok && b_ok)
            return round_like_ha_min_max((a + b) / 2.0f);
          if (a_ok)
            return round_like_ha_min_max(a);
          if (b_ok)
            return round_like_ha_min_max(b);
          return NAN;

```

