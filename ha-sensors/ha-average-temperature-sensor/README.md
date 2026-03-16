---
title: 'Define an Average Temperature sensor in Home Assistant'
date: '01-12-2024 13:14'
taxonomy:
    tag: [sensor, home assistant, template sensor]
metadata:
  description: This sensor calculates the average temperature of a zone; source sensors are specified in the sensor definition, without the need of a group or an extensive search. 
---

This sensor calculates the average temperature of a zone; source sensors are specified in the sensor definition, without the need of a group or an extensive search. 

===

## Code
```yaml
- platform: template
  sensors:
    zg_average_temperature:
      friendly_name: "Living Area Average Temperature"
      unit_of_measurement: "°C"
      value_template: >-
        {%- set sensors = ['sensor.temperatura_soggiorno_temperature', 'sensor.temperatura_cucina_temperature', 'sensor.mi_air_purifier_3_3h_temperature', 'sensor.temperatura_soggiorno_2_temperature'] -%}
        {%- set active_sensors = states|rejectattr('state','in',['unavailable','unknown','none']) | selectattr('entity_id', 'in', sensors) | map(attribute='state') | map('float') | list -%}
        {%- set active_count = active_sensors | count -%}
        {%- if active_count > 0 -%}
          {%- set average = active_sensors | sum / active_count-%}
          {{ average|round(1) }}
        {%- else -%}
          unavailable
        {%- endif -%}
```

## Description
This Home Assistant template sensor, **`zg_average_temperature`**, calculates the average temperature for a specified area (in this case, the living area). 
The benefit of an average temperature sensor is not only to measure a large area with more sensors, but also to create redundancy: infact, if a sensor is not available, you can use others to ensure the continuity of the automations based on that sensors (my heating system `climate` entity is based on this).
You can do the same with other numeric sensors, like humidity, pressure, and so on.
Here's how it works:

1. **Data Source**:  
   - The sensor pulls data from a predefined list of temperature sensors instead of a group (like I did in other examples):  
     `sensor.temperatura_soggiorno_temperature`, `sensor.temperatura_cucina_temperature`, `sensor.mi_air_purifier_3_3h_temperature`, and `sensor.temperatura_soggiorno_2_temperature`.

3. **Filtering Valid Sensors**:  
   - It excludes sensors with states like `unavailable`, `unknown`, or `none`.
   - The remaining sensors are converted to floating-point numbers for accurate calculations.

4. **Average Calculation**:  
   - If at least one valid sensor remains, it calculates the average temperature by summing the valid readings and dividing by their count, rounding the result to one decimal place.
   - If no valid sensors are available, it reports the state as `unavailable`.



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