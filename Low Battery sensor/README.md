# Define a Low Battery sensor in Home Assistant

This Home Assistant sensor, named **"Low Battery Devices"**, is a custom template sensor designed to list all devices with low battery levels. 
This sensor is useful for monitoring the battery status of devices in your smart home and identifying those requiring attention.
Unlike what I did in the Proxmox-Sinology backup sensor trick, you don't need to group all battery sensors.
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

## Enjoy
Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. All changes are made at your own responsibility. If this trick has been helpful, you can [!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoffee.com/moreno.sirri)
