# Smart Home Tricks
Welcome to my personal collection of Smart Home Tricks.

Here you can find some tricks related to the world of home automation, such as examples of complete use cases, or simple helpers, like sensor definitions or automations. 
Many examples take advantage of Home Assistant, since I have been using it for several years now, too.

<br/><br/>

## Home Assistant Sensor examples
Here you can find a couple of sensor definitions that you can use just as they are, or you can modify by your needs or simply take the inspiration to adapt the same technique into something else.

### [Open windows sensor](./Open%20Windows%20sensor/README.md)
- This sensor return the list of all open windows by checking all the `windows sensors` defined in a `group`.

### [Low battery sensor](./Low%20Battery%20sensor/README.md)
- This sensor return the list of all low battery devices. Unlike the previous one, it does not use groups but searches for all devices with certain attributes.

### [Average temperature sensor](./Average%20Temperature%20sensor/README.md)
- This sensor calculate the average temperature of a zone; source sensors are specified in the sensor definition, without the need of a group or an extensive search.

<br/><br/>

## Complete Use Cases
Here you can find some complete use cases, wich can be complex automations that involve more systems or technologies.

### [Proxmox and Sinology Backup status](./Proxmox-Sinology%20backup%20sensor/README.md)
- This process expose to Home Assistant the backup status (last successfull backup date) of **Proxmox VE**, **Sinology Active Backup for Business** and **Sinology Hyper Backup**.<br/> 
    In addition to `Proxmox VE`, `Sinology DSM` and `Home Assistant` it uses several tecnologies and products like `Node-RED`, `Gotify`, `MQTT`, `REST API`.

<br/><br/>

## Disclaimer
Even if I'll try to keep all this pages updated, products change over time, technologies evolve... so some use cases may no longer be necessary, some syntax may change, some technologies or products may no longer be available. Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar. <br/>
*Use these tricks under your own responsibility.*<br/>
If this trick has been helpful, you can <br/>[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoffee.com/moreno.sirri)
