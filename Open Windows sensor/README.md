# Define a Windows open sensor in Home Assistant

This Home Assistant sensor, named **"Open Windows"**, is a template sensor designed to track and report the state of windows in your home. Here's how it functions:

1. **Window Detection**:  
   It checks the entities in the group `group.windows` and filters for those with the state `on` (indicating an open window).

2. **Dynamic State Reporting**:  
   - If no windows are open, the sensor displays the message: *"The windows are closed."*
   - If one window is open, it specifies the name of that window, e.g., *"Living Room Window is open."*
   - If multiple windows are open, it lists all open windows, separating them with commas and adding "and" before the last one, e.g., *"Living Room Window, Bedroom Window, and Kitchen Window are open."*


This sensor is helpful for monitoring the open/closed status of your home's windows and providing a natural language summary of their state.

```yaml
template:
  - sensor:
    - unique_id: "1e186ee6108b450fbe0b60df5501ddd3"
      name: "Open Windows"
      icon: mdi:window
      state: >
        {% set open_windows = expand('group.windows') | selectattr('state','eq','on')
                            | map(attribute='name') | list %}
        {% if open_windows|length == 0 %}
            The windows are closed
        {% elif open_windows|length == 1 %}
            {{ open_windows[0] }} is open.
        {% else  %}
            {{ open_windows[:-1]|join(', ') }}{{ ', ' if open_windows|length > 2 else ' ' }} and {{ open_windows[-1] }}{{ ' are open.' if open_windows|length > 2 else ' are open.' }}
        {% endif %}

group:
  - windows:
    name: Finestre
    icon: mdi:window
    entities:
      - binary_sensor.portafinestra_cucina
      - binary_sensor.portafinestra_soggiorno
      - binary_sensor.finestra_camera
      - binary_sensor.finestra_seconda_stanza
      - binary_sensor.finestra_bagno
```

## Enjoy
Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. All changes are made at your own responsibility. If this trick has been helpful, you can [!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoffee.com/moreno.sirri)
