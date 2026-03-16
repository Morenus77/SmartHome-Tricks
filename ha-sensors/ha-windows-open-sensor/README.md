---
title: 'Define a Windows open sensor in Home Assistant'
date: '01-12-2024 12:08'
taxonomy:
    tag: [sensor, home assistant, template sensor]
metadata:
  description: This sensor returns the list of all open windows by checking all the windows sensors defined in a group. 
---

This sensor returns the list of all open windows by checking all the `windows sensors` defined in a `group`.

===

## Code
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

## Description
This Home Assistant sensor, named **"Open Windows"**, is a template sensor designed to track and report the state of windows in your home. Here's how it functions:

1. **Window Detection**:  
   It checks the entities in the group `group.windows` and filters for those with the state `on` (indicating an open window).

2. **Dynamic State Reporting**:  
   - If no windows are open, the sensor displays the message: *"The windows are closed."*
   - If one window is open, it specifies the name of that window, e.g., *"Living Room Window is open."*
   - If multiple windows are open, it lists all open windows, separating them with commas and adding "and" before the last one, e.g., *"Living Room Window, Bedroom Window, and Kitchen Window are open."*


This sensor is helpful for monitoring the open/closed status of your home's windows and providing a natural language summary of their state.



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