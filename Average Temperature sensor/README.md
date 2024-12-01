# Define an Average Temperature sensor in Home Assistant

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

## Enjoy
Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. All changes are made at your own responsibility. If this trick has been helpful, you can [!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoffee.com/moreno.sirri)
