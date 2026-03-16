# Smart Home Tricks
### Welcome to my personal collection of Smart Home Tricks.

Here you can find some tricks related to the world of home automation, such as examples of complete use cases, or simple helpers, like sensor definitions or automations. 
Many examples take advantage of Home Assistant, since I have been using it for several years now.

In addition you will find not only tricks related to the technical aspects of the smart home system, but also *tips on how technology can help us in our daily lives at home*.

<br/><br/>

## Home Assistant Sensor examples
Here you can find a couple of sensor definitions that you can use just as they are, or you can modify by your needs or simply take the inspiration to adapt the same technique into something else.

### [Average temperature sensor](/ha-sensors/ha-average-temperature-sensor/README.md)
This sensor calculate the average temperature of a zone; source sensors are specified in the sensor definition, without the need of a group or an extensive search.

### [Low battery sensor](/ha-sensors/ha-low-battery-sensor/README.md)
This sensor return the list of all low battery devices. Unlike the previous one, it does not use groups but searches for all devices with certain attributes.

### [Next Public Transport Departure sensor](/ha-sensors/ha-next-departure-sensor/README.md)
This sensor provides the waiting time in minutes for the next public transport, whether it’s a train, a bus, or — as in my case — a ferry.

### [Open windows sensor](/ha-sensors/ha-windows-open-sensor/README.md)
This sensor return the list of all open windows by checking all the `windows sensors` defined in a `group`.


<br/><br/>

## Complete Use Cases
Here you can find some complete use cases, wich can be complex automations that involve more systems or technologies.

### [AI Bill Assistant](/use-cases/ai-bill-assistant/README.md)
Forget about spending time to save and organize your bills, an AI based workflow will do it for you! This workflow read your bill related email, identify where the PDF bill is stored (it doesn't matter if attached to the email, linked in the email body or in a personal area website), extract main info (like the amount, reference period, etc) and save the PDF and also structured bill's data in a shared folder.<br/>
It uses `n8n`, `Open AI`, `REST API` and almost no programming.

### [AI Shopping Assistant](/use-cases/ai-shopping-assistant/README.md)
With this trick, whenever you find a recipe you like on a website, a PDF, an image on Instagram, or even in a message from a friend, all you need to do is share the content on Telegram. An AI-based bot will analyze the recipe, identify the ingredients, add them to your shopping list—preserving the quantities of those already on the list—and make them easily accessible on your smartphone or smartwatch for your next trip to the supermarket.<br/>
It uses `n8n`, `Open AI`, `REST API`, `Telegram Bot`, `Home Assistant`, `Bring!` and almost no programming.

### [Automatic Instagram Poster](/use-cases/automatic-instagram-poster/README.md)
Forget about spending time to manage your IG account (or Facebook page, Linkedin, ...): an AI based workflow will do it for you! This workflow takes the images you’ve edited and prepared for posting on Instagram, analyzes them, suggests the most suitable description and hashtags based on the content of each photo, and posts them on your behalf.
You have just to review the description and define the day you want to have each one published.<br/>
It uses `n8n`, `Open AI`, `Facebook REST API`, `Instagram`, `Facebook App`, `Google Drive` and just a very little `javascript` programming.

### [Expose Dockge data to Homepage](/use-cases/dockge-homepage/README.md)
This workaround allows **Homepage** to display real-time status information from **Dockge** container stacks, despite the absence of an official API, enabling centralized monitoring of your Docker containers.<br/>
It uses `Docker's native JSON export`, `Node-RED` for API transformation, and `Homepage's custom API widget` with minimal system overhead.

### [Proxmox resources monitoring](/use-cases/proxmox-resources-monitoring/README.md)
This process allows to expose in Home Assistant (or other Smart Home Hub) all information about all LXC and VMs defined in a **Proxmox VE** node.<br/>
It uses `Proxmox VE`, `Home Assistant`, `Node-RED`, `MQTT`, `MQTT Discovery`, `REST API` and a little `Javascript` programming.

### [Proxmox and Synology Backup status](/use-cases/proxmox-synology-backup/README.md)
This process expose to Home Assistant the backup status (last successfull backup date) of **Proxmox VE**, **Sinology Active Backup for Business** and **Sinology Hyper Backup**.<br/> 
In addition to `Proxmox VE`, `Sinology DSM` and `Home Assistant` it uses a little `Javascript` programming and several tecnologies and products like `Node-RED`, `Gotify`, `MQTT`, `REST API`.

### [Public Transport Next Departure Time and Status](/use-cases/public-transport-schedule/README.md)
With this workflow, every morning when I need to go to work, a widget on my smart home dashboard shows the waiting time for the next ferry departure, obtained from the official PDF schedule on the operator's website. Additionally, it monitors the municipality's official Telegram channel for any service interruptions due to weather conditions and notifies me both on the dashboard and via the voice assistant.<br/>
The entire process is optimized to minimize the use of AI and REST API calls.<br/>
It uses `n8n`, `Open AI`, `Home Assistant`, `Telegram`, `MQTT`, `REST API`, `Jinja2` and a bit of `javascript` programming.

### [Connect a Ring Video Doorbell camera in Frigate](/use-cases/ring-video-doorbell-frigate/README.md)
This process allows managing a **Ring Video Doorbell** (or another battery-powered camera) in **Frigate** or other NVR software that relies on continuous recording, without the risk of draining the battery.<br/>
It uses several tecnologies and products like `Frigate`, `Home Assistant`, `Scrypted`, and `REST API`.

### [Expose iOS Alarms to Home Assistant](/use-cases/sleep-alarm-home-assistant/README.md)
This simple process allows Home Assistant to automatically access the next alarm set on an **iPhone** (both standard alarm and wake-up one), making it available for automations like preheating the house or turning on the water heater in advance.<br/>
It uses only `Siri Shortcuts` and `Home Assistant` with just a bit of `Jinja2`templating.

<br/><br/>

## Disclaimer
Even if I'll try to keep all this pages updated, products change over time, technologies evolve... so some use cases may no longer be necessary, some syntax may change, some technologies or products may no longer be available. Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. <br/>
*Use these tricks under your own responsibility.*<br/>

<div class="myWrapper" style="text-align: center;" markdown="1">
If this tricks has been helpful, you can 

<a href="https://www.buymeacoffee.com/moreno.sirri" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
</div>


<sub>This work and all the contents of this repository are licensed under a **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
You can distribute, remix, adapt, and build upon the material in any medium or format, <u>for noncommercial purposes only by giving credit to the creator</u>. Modified or adapted material must be licensed under identical terms.
You can find the full license terms [here](LiCENSE)</sub>