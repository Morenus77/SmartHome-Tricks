---
title: 'Define a Low Batteries sensor in Home Assistant'
date: '01-12-2024 12:44'
taxonomy:
    tag:
        - sensor
        - 'home assistant'
        - 'template sensor'
metadata:
  description: This sensor returns the list of all low battery devices. Unlike the previous one, it does not use groups but searches for all devices with certain attributes. 
---

This sensor returns the list of all low battery devices. Unlike the previous one, it does not use groups but searches for all devices with certain attributes.

===

## Code
```yaml
template:
  - sensor:
    - unique_id: "8a3c846a-4141-4a0a-867d-857e7b8bc2a0"
      name: "Low Battery Devices"
      icon: mdi:battery-low
      state: >
        {% set threshold = states('input_number.battery_alarm_thresold') | int %}
        {%- set ns = namespace(sensors=[]) -%}
        {%- for state in states.sensor
            | selectattr('attributes.device_class', 'defined') 
            | selectattr('attributes.state_class', 'defined')
            | selectattr('attributes.device_class', '==', 'battery')
            | selectattr('attributes.state_class', '==', 'measurement')
            | selectattr('state', 'is_number') -%} 
            {%- if state.state | int <= threshold -%}
                {% set ns.sensors = ns.sensors + [dict(name = state.name | replace(' battery', '') | replace(' Battery',''), state = state.state | int)] %}
            {%- endif -%} 
        {%- endfor -%}
        {%- set batt = ns.sensors | sort(attribute='state') %}
        {%- set ns = namespace(batt='') -%}
        {%- for state in batt -%}
            {% set ns.batt = ns.batt + (state.name ~ ' (' ~ state.state ~ '%)' ~ "\n") %}
        {%- endfor -%}

        {% if ns.batt | count > 0 %}
            {{ ns.batt | truncate(255, true, '...') }}
        {% else %}
            {{ 'unavailable' }}
        {% endif %}

input_number:
  - battery_alarm_thresold:
    name: Soglia minima allarme batterie
    icon: mdi:battery-alert
    min: 1
    max: 50
    step: 1
```

## Description
This Home Assistant sensor, named **"Low Battery Devices"**, is a custom template sensor designed to list all devices with low battery levels. 
This sensor is useful for monitoring the battery status of devices in your smart home and identifying those requiring attention.
Unlike what I did in the Proxmox-Synology backup sensor trick, you don't need to group all battery sensors.
Here's how it works:

1. **Threshold Setting**:  
   It uses the value from the `input_number.battery_alarm_thresold` as the threshold for low battery levels.

2. **Device Selection**:  
   It scans all `sensor` entities with:
   - A `device_class` of `battery`.
   - A `state_class` of `measurement`.
   - A numeric state.

3. **Low Battery Check**:  
   If a sensor's battery level is at or below the threshold, it is included in the list.

4. **Formatted Output**:  
   - It generates a formatted list of device names and their battery levels, excluding the word "battery" from their names.  
   - The list is sorted by battery level (ascending).  
   - If the list is too long, it truncates to 255 characters.  
   - If no devices are found, it shows `unavailable`.


## Enjoy
Even if I'll try to keep all this pages updated, products change over time, technologies evolve... so some use cases may no longer be necessary, some syntax may change, some technologies or products may no longer be available. Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. <br/>
*Use this guide under your own responsibility.*<br/>

<div class="myWrapper" style="text-align: center;" markdown="1">
If this trick has been helpful, you can  <br/>

<a href="https://www.buymeacoffee.com/moreno.sirri" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
</div>

<br/>
<sub>This work and all the contents of this website are licensed under a **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
You can distribute, remix, adapt, and build upon the material in any medium or format, <u>for noncommercial purposes only by giving credit to the creator</u>. Modified or adapted material must be licensed under identical terms.
You can find the full license terms [here](https://creativecommons.org/licenses/by-nc-sa/4.0/?ref=chooser-v1)</sub>