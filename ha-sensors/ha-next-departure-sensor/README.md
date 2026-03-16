---
title: 'Define a Next Public Transport Departure sensor in Home Assistant'
date: '26-01-2025 14:00'
taxonomy:
    tag: [sensor, home assistant, template sensor, JSON]
metadata:
  description: This sensor provides the waiting time in minutes for the next public transport, whether it’s a train, a bus, or — as in my case — a ferry. It is based on a daily timetable in JSON, either set manually or obtained from a webpage, an API call, a PDF, or another data source. 
---

This sensor provides the waiting time in minutes for the next public transport, whether it’s a train, a bus, or — as in my case — a ferry. 
It is based on a daily timetable in JSON, either set manually or obtained from a webpage, an API call, a PDF, or another data source.

===

There are several ways to retrieve the daily schedule or directly the departure time of the next public transport (in which case this sensor would become unnecessary):
   - #### Specific API
This is the suggested method when the public transport operator directly provides its own set of APIs, but I won't cover it here because it is strictly tied to the specific case. Therefore, the API documentation of the transport operator must be consulted, and a **RESTful sensor** will likely need to be configured according to the official [Home Assistant guide](https://www.home-assistant.io/integrations/sensor.rest/).
   - #### Google Distance Matrix API
This method is perhaps the simplest, most effective, and suitable for almost any purpose, as most public transport is now recognized by Google Maps. 
It is also discussed in great detail in the documentation for the [Google Maps Travel Time integration](https://www.home-assistant.io/integrations/google_travel_time/), including the guide to obtain the API key from Google, so I don't think a further in-depth study is needed.
The guide also includes tips on how to optimize the use of the sensor without exceeding the free API usage quota and incurring charges.
   - #### Webpage scraping
You can use this method If Google Maps doesn't cover your local public transport (for example the ferry I use to go to work in summer) but it has the drawback that you need to work with **HTML** and **CSS**, and it is especially dependent on the structure of the page. Therefore, if the page changes in the future, the sensor will need to be reconfigured.
   - #### Webpage/PDF scraping with AI
This is the method I used in the **Public transport schedule and status** trick, so I suggest to read it for further info.
It uses AI to extract the schedule from a PDF and, early every morning, publishes the day's departures to an **MQTT topic**, which is mapped to a Home Assistant sensor. Another sensor - the one exposed in this article - read this daily schedule and return the next departure. Even though I could have directly returned the next scheduled departure, I chose this pattern to limit the number of API calls.

!!! In any case, even if it were possible to directly obtain the departure time of the next transport, for services with frequent departure times but schedules that do not change often, it might still make sense to retrieve the daily schedule only once a day. This approach would avoid excessive use of APIs and the risk of hitting limits or paying for exceeding free usage quotas. Using this sensor, you can then access this static information to determine the next departure time.

===

## Code
```yaml
template:
   - sensor:
      - unique_id: next_delfino_verde
        name: "Prossimo Delfino Verde"
        unit_of_measurement: "min"
        state: >
            {% set departures = states('sensor.delfino_verde_schedule') | from_json %}
            {% set now = now() %}
            {% set today = now.strftime('%Y-%m-%d') %}
            {% set tz = now.tzinfo %}
            {% set next_departure = departures 
                  | map('regex_replace', '^', today ~ ' ') 
                  | map('as_timestamp') 
                  | map('timestamp_local') 
                  | select('>', now | as_timestamp | timestamp_local) 
                  | list 
                  | first %}
            {% if next_departure %}
               {{ ((next_departure |as_datetime | as_local - now).seconds / 60) | int }}
            {% else %}
               -1
            {% endif %}

```

## Template Logic:

#### **Step 1: Extract the Schedule**
```jinja
{% set departures = states('sensor.delfino_verde_schedule') | from_json %}
```
- **`sensor.delfino_verde_schedule`**: This is another sensor that holds the ferry schedule as a JSON array with the departure times in `HH:mm` format, e.g., `["10:45", "11:55", "14:15", "15:45", "17:15", "18:45"]`. You can replace it with an `input_text` for static values.
- **`| from_json`**: Converts the JSON string into a list so it can be processed further.

---

#### **Step 2: Get the Current DateTime**
```jinja
{% set now = now() %}
{% set today = now.strftime('%Y-%m-%d') %}
{% set tz = now.tzinfo %}
```
- **`set now = now()`** retrieves the current date and time as a `datetime` object.
- **`set today = now.strftime('%Y-%m-%d')`** formats the current date into the format `YYYY-MM-DD` to prepend it to the departure times (e.g., `2025-01-19 10:45`).
- **`set tz = now.tzinfo`** retrieves the timezone information of the current time. This ensures all operations respect the local timezone.

---

#### **Step 3: Calculate the Next Departure**
```jinja
{% set next_departure = departures 
    | map('regex_replace', '^', today ~ ' ') 
    | map('as_timestamp') 
    | map('timestamp_local') 
    | select('>', now | as_timestamp | timestamp_local) 
    | list 
    | first %}
```

Here’s how it determines the next departure:
1. **`map('regex_replace', '^', today ~ ' ')`**:
   - Prepends today’s date (`YYYY-MM-DD`) to each departure time, converting `["10:45", "11:55"]` into `["2025-01-19 10:45", "2025-01-19 11:55"]`.

2. **`map('as_timestamp')`**:
   - Converts these datetime strings into UNIX timestamps.

3. **`map('timestamp_local')`**:
   - Ensures the timestamps are localized to the timezone.

4. **`select('>', now | as_timestamp | timestamp_local)`**:
   - Filters the departures to find those **later than the current time**.

5. **`list | first`**:
   - Converts the filtered departures into a list and picks the first (soonest) one.

---

#### **Step 4: Calculate Minutes Until Departure**
```jinja
{% if next_departure %}
  {{ ((next_departure |as_datetime | as_local - now).seconds / 60) | int }}
{% else %}
  -1
{% endif %}
```

1. **If a `next_departure` exists**:
   - Convert the timestamp of the next departure to a localized `datetime` object using `| as_datetime | as_local`.
   - Subtract the current time (`now`) from the next departure time.
   - Use `.seconds` to get the difference in seconds, divide by 60 to convert to minutes, and convert to an integer using `| int`.

2. **If no departures are found (`next_departure` is `None`)**:
   - Returns `-1` as a fallback value; It means that there are no more departures for the day.

---

### Example Walkthrough:

#### Input:
- **Current Time**: `2025-01-19 11:30`
- **JSON Schedule**: `["10:45", "11:55", "14:15", "15:45"]`

#### Steps:
1. **Prepend Today’s Date**:
   - `["2025-01-19 10:45", "2025-01-19 11:55", "2025-01-19 14:15", "2025-01-19 15:45"]`

2. **Convert to Timestamps**:
   - `["1705665900", "1705668900", "1705678500", "1705681500"]`

3. **Filter Future Departures**:
   - Future departures after `11:30`: `["1705668900", "1705678500", "1705681500"]`.

4. **Next Departure**:
   - The next departure is at `11:55` (`1705668900`).

5. **Calculate Time Difference**:
   - `11:55` - `11:30` = **25 minutes**.

#### Output:
- **Sensor Value**: `25`

---


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