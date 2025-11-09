# RSS and Home Assistant: early warning for grid blackouts

California, like most of the West Coast, is currently in the middle of a prolonged and serious heat-wave. Record breaking temperatures results in a distribution grid stressed beyond it's abilities which guarantees blackouts.

The [organization that oversees the electric grid](https://en.wikipedia.org/wiki/California_Independent_System_Operator) in California publishes RSS feeds for various types of grid related news and events. All the CA ISO RSS feeds are published [here](http://www.caiso.com/Pages/GlobalRSS.aspx), but the two feed that I'm using are:

- <http://content.caiso.com/awe/noticeRSS.xml>
- <http://content.caiso.com/awe/noticeflexRSS.xml>

As energy demand ramps up, `flex` alerts are issued as a call to reduce energy consumption. If demand continues to rise `warnings` and, finally, `stage3` events are issued. A `warning` does not guarantee a `stage3` event, but a `stage3` event mans that blackouts are all but inevitable in the coming hours. The *exact* meaning of each alert type is outlined in [this PDF](https://www.caiso.com/Documents/SystemAlertsWarningsandEmergenciesFactSheet.pdf).

The `noticeRSS.xml` feed, has *early* warning notifications that look like this:

```xml
<item>
\n
<title>NOTICE 202002440: CAISO Grid Warning Issued </title>
\n
<link>http://www.caiso.com/awe/noticelog.html#202002440</link>
\n
<description>Effective: 16-Aug-2020 12:00 PM Terminates: 16-Aug-2020 11:59 PM</description>
\n
</item>
```

Note: there's nothing wrong w/ your browser, the `\n` characters *are* in the actual feed. No matter, the [RSS parser](https://github.com/home-assistant/core/tree/dev/homeassistant/components/feedreader) in Home Assistant *can* tolerate them.

I use the alerts/warnings to do a few simple tasks, but there's so much potential beyond the automation I currently have:

- bumps thermostats up by a few degrees to lighten demand*
- re-schedule a few routine activities that are usually done in the evening to as soon as possible (cooking, for example)
- notify me to immediately go check that the generator has fuel

- Currently in 'unarmed' mode as I tweak the logic driving the decision

## How to do it

Basically, add the two feeds above to the [feedreader](https://www.home-assistant.io/integrations/feedreader/) integration and use the `feedreader` type of [event](https://www.home-assistant.io/docs/automation/trigger/#event-trigger) to trigger your automation.

Somewhere in `configuration.yaml`, load the `feedreader` component.

```yaml
feedreader: !include feedreader/rss.yaml

```

and in the `feedreader/rss.yaml` file, add the URLs:

```yaml
urls:
  - http://content.caiso.com/awe/noticeRSS.xml
  - http://content.caiso.com/awe/noticeflexRSS.xml
# Pull down the feed 2x/ hour
scan_interval:
  minutes: 30
```

Unfortunately, Home Assistant does not support setting the `scan_interval` per feed, so you'll have to figure out a happy middle ground if you've already got a few RSS feeds you monitor. Likewise, don't pull the feed down excessively... I don't see a lot of evidence that the feed is cached well and the CA ISO does occasionally have to put banners on the top of their site saying that the site is under heavy load. Don't be the reason the site goes down because *you* wanted **instant** notification.

If you're not in a position to do any sophisticated automation based on the notification, it's still simple to fire off a notification to *manually* prepare:

```yaml
##
- alias: Notify when new Flex or Warning issued
  trigger:
    platform: event
    event_type: feedreader
  action:
    service: persistent_notification.create
    data_template:
      notification_id: "{{ trigger.event.data.title }}"
      title: "FlexAlert"
      message: >
        {% set tokens = trigger.event.data.description.split(' ') %}
        {% set start_str = "{} {} {}".format(tokens[1], tokens[2], tokens[3]) %}
        {% set start = strptime(start_str, '%d-%b-%Y %I:%M %p') %}
        {% set end_str = "{} {} {}".format(tokens[5], tokens[6], tokens[7]) %}
        {% set end = strptime(end_str, '%d-%b-%Y %I:%M %p') %}
        "Heads up, electricity demand is likely to exceed production from {{ start.isoformat() }} to {{ end.isoformat() }}"
```

Which should create a simple notification that'll look something like this:

{{< figure name="example_notification" show_title="false" >}}

The `strptime` function yields `start` and `end` objects that can be manipulated further:

```jinja2
{% set now = now() %}
{% if start.day > now.day -%}
  It looks like demand will be high *tomorrow*
{%- endif %}
```

Happy automating.

