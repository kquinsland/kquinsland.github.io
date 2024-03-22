# Configuring additional ZwaveJS entities in Home Assistant over MQTT


This is a super quick "because the official docs didn't make it super clear so here's what ended up working for me" post.

-----
<!-- markdownlint-disable-file MD033 -->
After some [very disappointing WiFi connectivity issues]({{< relref "posts/2022/04/venstar-t7850-teardown-review#an-update-on-wifi-connectivity"  >}}), I settled on a Zwave based thermostat to replace the Venstar thermostat.

After installing the Honeywell TH6320 and connecting it to the ZwaveJS gateway, a new `climate`` entity appeared in Home Assistant. From there, I was able to see/control:

- The current thermostat setpoint/mode
- The current air temp and humidity
- The current battery level / if the thermostat thought the battery was low

I knew that it was possible to adjust the the screen backlight from the thermostat itself so I was a bit confused when there was no such configuration entity exposed in Home Assistant.

{{<figure name="ha_01">}}

After playing around with the ZwaveJS2MQTT web interface for bit, I discovered that it was possible to adjust the backlight brightness...and over 40 other settings under the `Configuration v4` section.

I can totally understand why only the "core" functionality would be automatically exposed to Home Assistant; most of the 40+ knobs are things that do not need to be set beyond the initial installation.

So, How do I go about getting ZwaveJS2MQTT to automatically tell Home Assistant about the current backlight level?

## Exposing 'idle brightness' to Home Assistant as a Sensor

From the `Configuration v4` tab for the thermostat node, I found the input field `[4-112-0-39] Idle Brightness` which let me control the thermostat screen backlight level.
I don't know what the numbers mean, but they are important / uniquely identify _a specific setting_.

{{< admonition note >}}
If you are using this as a guide but for a _different_ setting (perhaps you want to keep an eye on `[4-112-0-28] Minimum Cool Temperature`, for example) then you will want to substitute your parameter setting 'address' as appropriate.
{{< /admonition >}}

After some more experimentation and trying to grok [documentation](https://zwave-js.github.io/zwavejs2mqtt/#/homeassistant/homeassistant-mqtt?id=add-new-component), I figured out how to get ZwaveJS to advertise the current value for the backlight brightness as a sensor in Home Assistant.

{{<figure name="feat_zwjs2mqtt_01">}}

Here's how to do it:

- Select the `node` in question from the `Control Panel` page on the ZwaveJS2MQTT web UI. In this case, you can see that my thermostat is node `4`. Yours will likely be different.
- Select the `Home Assistant` tab from the left. You can see it selected in the above screenshot.
- In the blank `Hass Device JSON` box, paste the JSON document that describes the sensor / entity you wish to expose to Home Assistant. I struggled for some time trying to figure out what the document should look like but was able to 'reverse engineer' a working payload by studying the `sensor_humidity_air` entity that was automatically created.
- Then click the `ADD` button that should have activated just above the `Hass Device JSON`. A new row should appear in the `Home Assistant - Devices table`
- Click the newly added row so the `Hass Device JSON` field is populated with the JSON document you have added.
- Click the `UPDATE` button that should appear just above the `Hass Device JSON` label
- Assuming no issues, this will send the MQTT message which tells Home Assistant about your new sensor. You may wish to use a MQTT broker inspection app to monitor the exact payload that is sent to the `homeassistant/...` topic as well as `tail -f home-assistant.log`. If Home Assistant gets the message, any validation errors will show up in the HA log.
- Once you are satisfied with how the entity is presented in Home Assistant, click the `STORE` button just below the `Home Assistant - Devices` heading on the ZwaveJS2MQTT web UI.

When all is said and done, this is what you should see:

{{<figure name="zwjs2mqtt_02">}}

### Example payload

Here is a (slightly edited) version of the JSON document I added via the `Hass Device JSON` entry.

```json
{
  "type": "sensor",
  "object_id": "idle_brightness",
  "discovery_payload": {
    "value_template": "{{ value_json.value }}",
    "state_topic": "zwave/YourThermostat_NameHere/112/0/39",
    "json_attributes_topic": "zwave/YourThermostat_NameHere/112/0/39",
    "device": {
      "identifiers": [
        "zwavejs2mqtt_0xdeadbeef_node4"
      ],
      "manufacturer": "Honeywell",
      "model": "T6 Pro Z-Wave Programmable Thermostat (TH6320ZW)",
      "name": "YourThermostat NameHere",
      "sw_version": "1.3"
    },
    "name": "YourThermostat NameHere idle_brightness",
    "unique_id": "zwavejs2mqtt_0xdeadbeef_4-112-0-Idle_Brightness",
    "entity_category": "config",
    "icon": "mdi:brightness-percent"
  },
  "discoveryTopic": "sensor/YourThermostat_NameHere/idle_brightness/config",
  "values": [
    "112-0-39"
  ],
  "persistent": false,
  "ignoreDiscovery": false,
  "id": "sensor_idle_brightness"
}
```

I have replaced a few things in the example payload. You will want to retrieve the correct values for your payload from one of the automatically generated payloads. I used the `sensor_humidity_air` device as a starting point for my payload.

## Ok, but what about writing to the thermostat?

{{<figure name="ha_02">}}

Once the sensor exists in Home Assistant, you will probably want to use it in an automation.
Here is a simple automation that watches a basic number input widget and sets the backlight brightness for the thermostat based on what the widget is set to:

```yaml
alias: Sync thermostat brightness to input select
description: An Example automation.
trigger:
  - platform: state
    entity_id:
      - sensor.yourthermostat_namehere_idle_brightness
    id: physical
  - platform: state
    entity_id:
      - input_number.thermostat_backlight
    id: virtual
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id: virtual
        sequence:
          - service: mqtt.publish
            data:
              topic: zwave/YourThermostat_NameHere/112/0/39/set
              payload_template: '{{trigger.to_state.state}}'
      - conditions:
          - condition: trigger
            id: physical
        sequence:
          - service: input_number.set_value
            target:
              entity_id: input_number.thermostat_backlight
            data_template:
              value: '{{trigger.to_state.state | float}}'
    default: []
mode: single
max_exceeded: silent
```

This _works_ but it's not _automagic_. We can do better!

## More Automagic, More Better

The primary downside with the above automation is that the user (read: you) needs to create the `input_number` widget and then spend the time telling Home Assistant what to do when the value changes.

This isn't a particularly difficult task - only ~30 lines of yaml... but what if we didn't have to do that?

What if Home Assistant had a robust auto discovery mechanism that we could take advantage of to _automagically_ set up a basic input widget that would transmit the necessary MQTT payload to adjust the backlight ... automatically?

I have good news and bad news.

The good news is that we totally can do this. 🎉

The bad news is that ZwaveJS2MQTT does not appear to support the [`input_select`](https://www.home-assistant.io/integrations/input_select/) type of entity that we'll need to pull this off. 👎

I am not super familiar with the ZwaveJS2MQTT code base, but it looks like there is no validation on the JSON you enter under the `Hass Device JSON` field but there _is_ some validation / filtering on the MQTT message that is sent for auto discovery. [ZwaveJS2MQTT does not seem to support `select` components via MQTT](https://github.com/zwave-js/zwavejs2mqtt/blob/master/hass/configurations.ts#L6) so the payload that is sent to Home Assistant is incomplete and results in errors:

```shell
│ ERROR (MainThread) [homeassistant.util.logging] Exception in discovery_callback when dispatching 'mqtt_discovery_updated_('select', 'YourThermostat NameHere idle_brightness │
│ Traceback (most recent call last):
│   File "/usr/src/homeassistant/homeassistant/components/mqtt/mixins.py", line 724, in discovery_callback      
│     await self._discovery_update(payload)    
│   File "/usr/src/homeassistant/homeassistant/components/mqtt/mixins.py", line 886, in discovery_update        
│     config = self.config_schema()(discovery_payload)                                                          
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/validators.py", line 232, in __call__               
│     return self._exec((Schema(val) for val in self.validators), v)                                            
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/validators.py", line 355, in _exec                  
│     raise e if self.msg is None else AllInvalid(self.msg, path=path)                                          
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/validators.py", line 351, in _exec                  
│     v = func(v)                              
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/schema_builder.py", line 272, in __call__           
│     return self._compiled([], data)          
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/schema_builder.py", line 818, in validate_callable  
│     return schema(data)                      
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/schema_builder.py", line 272, in __call__           
│     return self._compiled([], data)          
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/schema_builder.py", line 595, in validate_dict      
│     return base_validate(path, iteritems(data), out)                                                          
│   File "/usr/local/lib/python3.9/site-packages/voluptuous/schema_builder.py", line 433, in validate_mapping   
│     raise er.MultipleInvalid(errors)         
│ voluptuous.error.MultipleInvalid: required key not provided @ data['options']   
```

While it would be nice if ZwaveJS2MQTT would support more device types, we have a pretty simple fix: use another device to publish a valid configuration payload.

And as it turns out, it's _trivial_ to get Home Assistant to publish an arbitrary payload to an arbitrary topic when it starts up.

```yaml
alias: Autoconfigure Thermostat Backlight select
description: >-
  On HA start up, publish MQTT payload so HA auto-discovers the input_select to
  automate thermostat backlight brightness
trigger:
  - platform: homeassistant
    event: start
condition: []
action:
  - service: mqtt.publish
    data:
      topic: homeassistant/select/YourThermostat_NameHere/idle_brightness/config
      payload: >-
        {"options":["0","1","2","3","4","5"],"name":"MainThermostat idle_brightness","state_topic":"zwave/YourThermostat_NameHere/112/0/39","value_template":"{{'{{value_json.value}}'}}","command_topic":"zwave/YourThermostat_NameHere/112/0/39/set","command_template":"{{'{{value}}'}}","unique_id":"zwavejs2mqtt_0xdeadbeef_4-112-0-Idle_Brightness_Select","device":{"identifiers":["zwavejs2mqtt_0xdeadbeef_node4"],"manufacturer":"Honeywell","model":"T6ProZ-WaveProgrammableThermostat(TH6320ZW)","name":"MainThermostat","sw_version":"1.3"},"entity_category":"config","icon":"mdi:brightness-percent"}
mode: single
```

{{< admonition important "Notice" >}}
If you look carefully, you will notice two things:

- The json payload has been escaped and turned into a string
- The templates inside the string are [_further_ escaped](https://community.home-assistant.io/t/how-can-i-use-escape-characters-while-templating/135324) by wrapping them in `{{ '` and `'}}`.

{{< /admonition >}}

Here is a pretty formatted JSON document:

```json
{
   "options":[
      "0",
      "1",
      "2",
      "3",
      "4",
      "5"
   ],
   "name":"YourThermostat NameHere idle_brightness",
   "state_topic":"zwave/YourThermostat_NameHere/112/0/39",
   "value_template":"{{ '{{value_json.value}}' }}",
   "command_topic":"zwave/YourThermostat_NameHere/112/0/39/set",
   "command_template":"{{ '{{value}}' }}",
   "unique_id":"zwavejs2mqtt_0xdeadbeef_4-112-0-Idle_Brightness_Select",
   "device":{
      "identifiers":[
         "zwavejs2mqtt_0xdeadbeef_node4"
      ],
      "manufacturer":"Honeywell",
      "model":"T6 Pro Z-Wave Programmable Thermostat (TH6320ZW)",
      "name":"YourThermostat NameHere",
      "sw_version":"1.3"
   },
   "entity_category":"config",
   "icon":"mdi:brightness-percent"
}
```

Once the automation is created, save it. You can test your work by clicking "run actions" and monitoring the home assistant log file.

When the payload is properly escaped, you should see a new entity added to the Device page:

{{<figure name="ha_03">}}

When you select a brightness value from the drop down, the backlight brightness should change.
That's how you get a 'non standard' thermostat configuration value to automatically show up on the correct device page.

You now no longer need the example automation from [above](#ok-but-what-about-writing-to-the-thermostat). 😎.

