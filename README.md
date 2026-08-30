# Smart Home Tricks
### Welcome to my personal collection of Smart Home Tricks.

<div style="text-align: center"><img src="smart-home-tricks-logo.webp" width=50% height=50%></div>

Here you can find some tricks related to the world of home automation, such as examples of complete use cases, or simple helpers, like sensor definitions or automations.
Many examples take advantage of Home Assistant, since I have been using it for several years now.

In addition you will find not only tricks related to the technical aspects of the smart home system, but also *tips on how technology can help us in our daily lives at home*.

[![View on SmartHomeTricks](https://img.shields.io/badge/View_on-SmartHomeTricks-blue?style=for-the-badge&logo=homeassistant&logoColor=white)](https://smarthometricks.sirri.it)

<br/><br/>

## Sensors, packages & simple automations
Here you can find a couple of sensor definitions, packages or simple automations that you can use just as they are, or you can modify by your needs or simply take the inspiration to adapt the same technique into something else.

### [Automated House Sitter Access Management in Home Assistant](/ha-sensors/ha-housesitter-package/README.md)
This package creates a full automatic system for managing temporary access for house sitters, cleaners, or maintenance workers in Home Assistant.

<sub>**Date:** 01/05/2026<br/>**Tags:** package, home assistant, template sensor, lovelace, automation</sub>

<br/>

### [Define a Next Public Transport Departure sensor in Home Assistant](/ha-sensors/ha-next-departure-sensor/README.md)
This sensor provides the waiting time in minutes for the next public transport, whether it's a train, a bus, or — as in my case — a ferry.

<sub>**Date:** 26/01/2025<br/>**Tags:** sensor, home assistant, template sensor, JSON</sub>

<br/>

### [Define an Average Temperature sensor in Home Assistant](/ha-sensors/ha-average-temperature-sensor/README.md)
This sensor calculate the average temperature of a zone; source sensors are specified in the sensor definition, without the need of a group or an extensive search.

<sub>**Date:** 01/12/2024<br/>**Tags:** sensor, home assistant, template sensor</sub>

<br/>

### [Define a Low Batteries sensor in Home Assistant](/ha-sensors/ha-low-battery-sensor/README.md)
This sensor return the list of all low battery devices. Unlike the previous one, it does not use groups but searches for all devices with certain attributes.

<sub>**Date:** 01/12/2024<br/>**Tags:** sensor, home assistant, template sensor</sub>

<br/>

### [Define a Windows open sensor in Home Assistant](/ha-sensors/ha-windows-open-sensor/README.md)
This sensor return the list of all open windows by checking all the `windows sensors` defined in a `group`.

<sub>**Date:** 01/12/2024<br/>**Tags:** sensor, home assistant, template sensor</sub>

<br/>

<br/><br/>

## Complete Use Cases
Here you can find some complete use cases, wich can be complex automations that involve more systems or technologies.

### [Build a personal AI for your home — with a private knowledge base](/use-cases/personal-ai-private-knowledge-base/README.md)
An AI Agent that orchestrates three very different kinds of knowledge — Home Assistant as a delegated third-party system, a vector store over your static private documents, and a realtime API over you...<br/>
It uses `n8n`, `Open AI`, `Home Assistant`, `Vector Store`, `REST API` and a bit of `javascript` programming.

<sub>**Date:** 16/08/2026<br/>**Tags:** Open AI, n8n, Telegram Bot, Home Assistant, Proxmox VE, RAG, REST API, Claude AI</sub>

<br/>

### [Query your bills and documents with a private RAG knowledge base](/use-cases/bills-documents-rag-knowledge-base/README.md)
After the AI Bill Assistant files my bills and the Claude Code skill do the same for sporadic documents, the numbers were still stuck in folders, even if in a structured format. So I built a private R...<br/>
It uses `n8n`, `Open AI`, `RAG`, `Vector Store` and a bit of `javascript` programming.

<sub>**Date:** 15/08/2026<br/>**Tags:** Open AI, n8n, RAG, Qdrant, Vector Store, Chatbot, REST API, Claude AI, Mongo DB, OCR</sub>

<br/>

### [Scan and analyze documents with Claude Code](/use-cases/scan-analyze-documents-claude-code/README.md)
A workshop service record, an inspection, the annual vehicle tax, an occasional receipt or contract — sporadic documents with no fixed layout never justified a dedicated n8n workflow. So I built a Cla...<br/>
It uses `Claude AI`, `OCR`, `Mongo DB`, `REST API` and minimal configuration.

<sub>**Date:** 14/08/2026<br/>**Tags:** Claude AI, OCR, Mongo DB, macOS, Vision, Swift</sub>

<br/>

### [Proxmox HomeLab status monitoring](/use-cases/proxmox-homelab-status/README.md)
I keep an eye on my whole Proxmox cluster without staring at dashboards: an n8n workflow reads the Proxmox API natively, walks nodes, VMs and LXC containers one by one, checks everything and messages ...<br/>
It uses `n8n`, `Proxmox VE`, `Telegram Bot` and minimal configuration.

<sub>**Date:** 09/08/2026<br/>**Tags:** Proxmox VE, n8n, Telegram Bot, REST API, Javascript, n8n-demo</sub>

<br/>

### [House Sitter Chatbot](/use-cases/housesitter-chatbot/README.md)
Forget having to be available when the house sitter arrives to open the gate and front door for her/him… a virtual butler will welcome her/him in your place, managing security and protecting you from ...<br/>
It uses `Claude AI`, `Telegram Bot`, `Chatbot` and minimal configuration.

<sub>**Date:** 10/05/2026<br/>**Tags:** Claude AI, Telegram, Chatbot, Home Assistant, n8n, Mongo DB, REST API</sub>

<br/>

### [Expose Dockge data to Homepage](/use-cases/dockge-homepage/README.md)
This workaround allows Homepage to display real-time status information from Dockge container stacks, despite the absence of an official API, enabling centralized monitoring of your Docker containers.<br/>
It uses `Docker's native JSON export`, `Node-RED` for API transformation, and `Homepage's custom API widget` with minimal system overhead.

<sub>**Date:** 03/11/2025<br/>**Tags:** Docker, Node-RED, Homepage, REST API, Home Assistant</sub>

<br/>

### [Expose iOS Alarms to Home Assistant](/use-cases/sleep-alarm-home-assistant/README.md)
This simple process allows Home Assistant to automatically access the next alarm set on an iPhone (both standard alarm and wake-up one), making it available for automations like preheating the house o...<br/>
It uses only `Siri Shortcuts` and `Home Assistant` with just a bit of `Jinja2` templating.

<sub>**Date:** 08/02/2025<br/>**Tags:** IOS, Siri Shortcuts, Home Assistant, Jinja2</sub>

<br/>

### [Public Transport Next Departure Time and Status](/use-cases/public-transport-schedule/README.md)
With this workflow, every morning when I need to go to work, a widget on my smart home dashboard shows the waiting time for the next ferry departure, obtained from the official PDF schedule on the ope...<br/>
It uses `n8n`, `Open AI`, `Home Assistant`, `Telegram`, `MQTT`, `REST API`, `Jinja2` and a bit of `javascript` programming.

<sub>**Date:** 26/01/2025<br/>**Tags:** Open AI, n8n, MQTT, REST API, Javascript, Telegram Bot, Jinja2, Home Assistant</sub>

<br/>

### [AI Shopping Assistant](/use-cases/ai-shopping-assistant/README.md)
With this trick, whenever you find a recipe you like on a website, a PDF, an image on Instagram, or even in a message from a friend, all you need to do is share the content on Telegram. An AI-based bo...<br/>
It uses `n8n`, `Open AI`, `REST API`, `Telegram Bot`, `Home Assistant`, `Bring!` and almost no programming.

<sub>**Date:** 06/01/2025<br/>**Tags:** Open AI, n8n, REST API, Telegram Bot, Home Assistant, Bring!</sub>

<br/>

### [Automatic Instagram Posting](/use-cases/automatic-instagram-poster/README.md)
Forget about spending time to manage your IG account (or Facebook page, Linkedin, ...): an AI based workflow will do it for you! This workflow takes the images you’ve edited and prepared for posting o...<br/>
It uses `n8n`, `Open AI`, `Facebook REST API`, `Instagram`, `Facebook App`, `Google Drive` and just a very little `javascript` programming.

<sub>**Date:** 03/01/2025<br/>**Tags:** Open AI, n8n, Facebook REST API, Instagram, Facebook App, Google Drive, Javascript</sub>

<br/>

### [AI Bill Assistant](/use-cases/ai-bill-assistant/README.md)
Forget about spending time to save and organize your bills, an AI based workflow will do it for you! This workflow read your bill related email, identify where the PDF bill is stored (it doesn't matte...<br/>
It uses `n8n`, `Open AI`, `REST API` and almost no programming.

<sub>**Date:** 25/12/2024<br/>**Tags:** Open AI, n8n, REST API</sub>

<br/>

### [Proxmox resources monitoring](/use-cases/proxmox-resources-monitoring/README.md)
This process allows to expose in Home Assistant (or other Smart Home Hub) all information about all LXC and VMs defined in a Proxmox VE node.<br/>
It uses `Proxmox VE`, `Home Assistant`, `Node-RED`, `MQTT`, `MQTT Discovery`, `REST API` and a little `Javascript` programming.

<sub>**Date:** 14/12/2024<br/>**Tags:** Proxmox VE, Home Assistant, Node-RED, MQTT, MQTT Discovery, REST API, Javascript</sub>

<br/>

### [Connect a Ring Video Doorbell camera in Frigate](/use-cases/ring-video-doorbell-frigate/README.md)
This process allows managing a Ring Video Doorbell (or another battery-powered camera) in Frigate or other NVR software that relies on continuous recording, without the risk of draining the battery.<br/>
It uses several tecnologies and products like `Frigate`, `Home Assistant`, `Scrypted`, and `REST API`.

<sub>**Date:** 03/12/2024<br/>**Tags:** Frigate, Home Assistant, Scrypted, REST API</sub>

<br/>

### [Proxmox and Synology Backup status](/use-cases/proxmox-synology-backup/README.md)
This workflow exposes to Home Assistant the backup status (last successfull backup date) of Proxmox VE, Synology Active Backup for Business and Synology Hyper Backup.<br/>
In addition to `Proxmox VE`, `Synology DSM` and `Home Assistant` it uses a little `Javascript` programming and several tecnologies and products like `Node-RED`, `Gotify`, `MQTT`, `REST API`.

<sub>**Date:** 30/11/2024<br/>**Tags:** Proxmox VE, Home Assistant, Template sensor, Lovelace, Synology DSM, MQTT, Node-RED, Gotify, REST API, Javascript</sub>

<br/>

<br/><br/>

## Disclaimer
Even if I'll try to keep all this pages updated, products change over time, technologies evolve... so some use cases may no longer be necessary, some syntax may change, some technologies or products may no longer be available. Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. <br/>
*Use these tricks under your own responsibility.*<br/>

<div class="myWrapper" style="text-align: center;" markdown="1">
If this tricks has been helpful, you can 

<a href="https://www.buymeacoffee.com/moreno.sirri" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
</div>


<sub>Read all the tricks with images, diagrams and the full interactive version on **SmartHomeTricks**: [https://smarthometricks.sirri.it](https://smarthometricks.sirri.it)</sub>


<sub>This work and all the contents of this repository are licensed under a **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
You can distribute, remix, adapt, and build upon the material in any medium or format, <u>for noncommercial purposes only by giving credit to the creator</u>. Modified or adapted material must be licensed under identical terms.
You can find the full license terms [here](LICENSE)</sub>
