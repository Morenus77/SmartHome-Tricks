---
title: 'AI Shopping Assistant'
published: true
date: '06-01-2025 13:00'
media_order: ai-shopping-assistant.webp
taxonomy:
    tag:
        - 'Open AI'
        - n8n
        - 'REST API'
        - 'Telegram Bot'
        - 'Home Assistant'
        - Bring!
image_url: 'https://smarthometricks.sirri.it/user/pages/01.home/_use-cases/ai-shopping-assistant/ai-shopping-assistant.webp'
metadata:
    'og:title': 'AI shopping assistant'
    'og:type': article
    'og:image': 'https://smarthometricks.sirri.it/user/pages/01.home/_use-cases/ai-shopping-assistant/preview.webp'
    'og:url': 'https://smarthometricks.sirri.it/home/_use-cases/ai-shopping-assistant'
    'og:description': 'With this trick, whenever you find a recipe you like on a website, a PDF, an image on Instagram, or even in a message from a friend, all you need to do is share the content on Telegram. An AI-based bot will analyze the recipe, identify the ingredients, add them to your shopping list—preserving the quantities of those already on the list—and make them easily accessible on your smartphone or smartwatch for your next trip to the supermarket.'
    description: 'With this trick, whenever you find a recipe you like on a website, a PDF, an image on Instagram, or even in a message from a friend, all you need to do is share the content on Telegram. An AI-based bot will analyze the recipe, identify the ingredients, add them to your shopping list—preserving the quantities of those already on the list—and make them easily accessible on your smartphone or smartwatch for your next trip to the supermarket.'
sitemap:
    lastmod: '31-05-2025 09:27'
---

With this trick, whenever you find a recipe you like on a website, a PDF, an image on Instagram, or even in a message from a friend, all you need to do is share the content on Telegram. An AI-based bot will analyze the recipe, identify the ingredients, add them to your shopping list—preserving the quantities of those already on the list—and make them easily accessible on your smartphone or smartwatch for your next trip to the supermarket.<br/>
It uses `n8n`, `Open AI`, `REST API`, `Telegram Bot`, `Home Assistant`, `Bring!` and almost no programming.

===

<script src="https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@2.0.0/webcomponents-loader.js"></script>
<script src="https://www.unpkg.com/lit@2.0.0-rc.2/polyfill-support.js"></script>
<script type="module" src="https://cdn.jsdelivr.net/npm/@n8n_io/n8n-demo-component/n8n-demo.bundled.js"></script>

{assets:inline_css}
n8n-demo {
	--n8n-workflow-min-height: 500px;
}
{/assets}

# AI shopping assistant

Okay, I have to admit it, this trick was my wife's idea 😊

We use **[Bring!](https://www.getbring.com/en/home)** as our shared shopping list app. Thanks to the **Apple Watch** app, when we're at the supermarket, we can simply glance at our wrist to see the items on the list, and a single tap removes items we've added to the cart. To add items to the list, besides using the app on our smartphones, we use a voice assistant when we're at home. It's much more immediate since we can activate it the moment we realize we need something.

Unfortunately, for the past few months, **Amazon Echo** has no longer allowed the use of third-party apps to manage lists. So instead of saying, "Alexa, add tomatoes to the shopping list," we now say, "Alexa, open Bring and add tomatoes" (saying "to the shopping list" is redundant here because we only have one list). It's a minor inconvenience, but manageable.

Every now and then, we share recipes we find online, in our cooking robot's app, or on Instagram. But we almost always forget to save the ingredients and add them to the shopping list. Even if we do remember, it’s a tedious and repetitive task—manually adding each ingredient along with its quantity. So, we thought: why not let AI handle it?

## Process Description
- Whenever we come across a recipe — whether it's a link to a website, a screenshot from a video, a PDF document or a message from friends — we share it on our dedicated **Telegram channel**.
- A bot analyzes the shared content and, using AI, extracts the ingredients from the recipe. It then checks which items are already on the list to update their quantities and adds any new items.
- The Bring! app on our smartphones or Apple Watch is automatically synced with the updated or newly added items.

![workflow](ai-shopping-assistant.webp)

<br/><br/>


# Step 1: Create Telegram Bot
I chose **Telegram** for two reasons:
- One of my wife’s prerequisites was "no new apps" (we’re technologically complementary 😅)
- Telegram is easy to use for sharing from different apps and allows you to delete anything that’s no longer needed.

<br/>

Creating a **Telegram Bot** is really really simple:
- Open a chat with the user **@BotFather** and type `/newbot`
- Follow the instructions: you will be asked to define a **Bot name** and **username**
- At the end you will see the **HTTP API Token** (`[Telegram_Token]`): save it securely and we'll use it later to configure the Telegram trigger of the workflow

<br/><br/>

# Step 2: Configure Home Assistant

## Introduction
kay, I have to admit, I left out a small detail—sorry about that! I didn't directly use Bring's official APIs. In fact, I couldn't find any mention in the official documentation of interacting with a specific shopping list to add, modify, or remove items. The only supported functionality seems to be submitting a recipe and letting Bring's scraper extract the ingredients. However, the recipe must be in a specific format, likely because Bring doesn't use AI for scraping, as I did, but instead relies on an algorithm based on a well-defined data structure.

That said, there is a way to interact with Bring using unofficial APIs. In my case, I didn’t spend time researching this further since I had already integrated Bring! into  **Home Assistant**, which made it easy to use HA as a "proxy."

You're not obligated to use Home Assistant: if you can find the documentation for the unofficial APIs (I found sometrhing on GitHub but don't post the link since it may chance), you just need to modify the steps in the following workflow, replacing the Home Assistant REST APIs with Bring's APIs.

Moreover, my ultimate goal is to replace Amazon Echo as a voice assistant with one (ideally local) powered by AI. In such a scenario, this assistant would interact directly with the **To Do List** in Home Assistant.


## Integrate Bring into Home Assistant
The setup is very straightforward thanks to the [Official Bring! Integration](https://www.home-assistant.io/integrations/bring). Once configured by logging into Bring! with your account, the lists you’ve set up in Bring! will appear as **To Do List**  in Home Assistant. For instance, in my case, `todo.spesa`.

Now, any changes to the `todo.spesa` list will immediately sync with the Bring! app and vice versa.

I'm not showing you how to set up Bring on Amazon Echo or Google Home because — even if it's useful for everyday use — it's not a fundamental prerequisite for this process.

## Configure Home Assistant API
Very simple: open your `configuration.yaml` and add the line:
``` yaml
api:
```
Now you can access to Home Assistant RESTful API calling whe address `http[s]://[Home_Assistant_IP]:[port]/api/`.
Remember to set the `http` or `https` procotol accordingly to your setup, and the correct IP and port.

## Obtain a Long-Lived Access Token
Log into the frontend using a web browser, go to your profile and under "Security" menu you wull be able to list, create and revoke all the Long-Lived Token associated with your account.


<br/><br/>

# Step 3: Install and configure n8n
As with other complex automations you will find on this site, I used **n8n**: n8n is an open-source workflow automation tool that allows you to connect various applications and services to automate repetitive tasks without manual intervention. It provides a visual interface where you can design workflows by linking different nodes that represent actions, triggers, or data processing steps. 

I installed it in a Proxmox LXC by using the helper script provided by <a href="https://community-scripts.github.io/ProxmoxVE">Community-Scripts</a>: this is a very useful Community with many script that will help you many times: if you like them, consider donating to support Angie, tteckster's wife - the founder and best supporter of the community - too early passed away.

<br/><br/>

# Step 4: Create the automation flow

This **n8n workflow** integrates a Telegram bot with Home Assistant and uses AI for natural language processing to manage a shopping list. Here's an overview:

<n8n-demo workflow='%7B%22nodes%22%3A%5B%7B%22parameters%22%3A%7B%22options%22%3A%7B%7D%7D%2C%22id%22%3A%222841152b-3259-49a8-baaa-5bcbaf049967%22%2C%22name%22%3A%22OpenAI%20Chat%20Model%22%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.lmChatOpenAi%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B160%2C740%5D%2C%22credentials%22%3A%7B%22openAiApi%22%3A%7B%22id%22%3A%2269QZlxxBRyU7gp8p%22%2C%22name%22%3A%22OpenAi%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22schemaType%22%3A%22manual%22%2C%22inputSchema%22%3A%22%7B%5Cn%20%20%5C%22type%5C%22%3A%20%5C%22array%5C%22%2C%5Cn%20%20%5C%22items%5C%22%3A%20%7B%5Cn%20%20%20%20%5C%22type%5C%22%3A%20%5C%22object%5C%22%2C%5Cn%20%20%20%20%5C%22properties%5C%22%3A%20%7B%5Cn%20%20%20%20%20%20%5C%22ingredient%5C%22%3A%20%7B%20%5C%22type%5C%22%3A%20%5C%22string%5C%22%20%7D%2C%5Cn%20%20%20%20%20%20%5C%22quantity%5C%22%3A%20%7B%20%5C%22type%5C%22%3A%20%5C%22string%5C%22%20%7D%5Cn%20%20%20%20%7D%5Cn%20%20%7D%5Cn%7D%5Cn%5Cn%22%7D%2C%22id%22%3A%22715e2143-faa1-4441-9068-31eec92b856e%22%2C%22name%22%3A%22Structured%20Output%20Parser1%22%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.outputParserStructured%22%2C%22typeVersion%22%3A1.2%2C%22position%22%3A%5B480%2C740%5D%7D%2C%7B%22parameters%22%3A%7B%22mode%22%3A%22combine%22%2C%22advanced%22%3Atrue%2C%22mergeByFields%22%3A%7B%22values%22%3A%5B%7B%22field1%22%3A%22ingredient%22%2C%22field2%22%3A%22summary%22%7D%5D%7D%2C%22joinMode%22%3A%22keepNonMatches%22%2C%22outputDataFrom%22%3A%22input1%22%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.merge%22%2C%22typeVersion%22%3A3%2C%22position%22%3A%5B1180%2C680%5D%2C%22id%22%3A%225512a761-d725-4f58-becd-a8a20c25f158%22%2C%22name%22%3A%22Get%20new%20items%22%7D%2C%7B%22parameters%22%3A%7B%22fieldToSplitOut%22%3A%22service_response%5B%27todo.spesa%27%5D.items%22%2C%22options%22%3A%7B%7D%7D%2C%22id%22%3A%2256004c1a-4ddd-4eee-86c3-1ac1622785dc%22%2C%22name%22%3A%22List%20current%20items%22%2C%22type%22%3A%22n8n-nodes-base.itemLists%22%2C%22position%22%3A%5B900%2C900%5D%2C%22notesInFlow%22%3Afalse%2C%22typeVersion%22%3A1%2C%22notes%22%3A%22Create%20Items%20from%20Body%22%7D%2C%7B%22parameters%22%3A%7B%22fieldToSplitOut%22%3A%22output%22%2C%22options%22%3A%7B%7D%7D%2C%22id%22%3A%229ad1ae2f-09fb-4f8b-a067-4760e37f550b%22%2C%22name%22%3A%22List%20new%20items%22%2C%22type%22%3A%22n8n-nodes-base.itemLists%22%2C%22position%22%3A%5B900%2C220%5D%2C%22notesInFlow%22%3Afalse%2C%22typeVersion%22%3A1%2C%22notes%22%3A%22Create%20Items%20from%20Body%22%7D%2C%7B%22parameters%22%3A%7B%22method%22%3A%22POST%22%2C%22url%22%3A%22https%3A%2F%2F%5BHome_Assistant_IP%5D%3A%5Bport%5D%2Fapi%2Fservices%2Ftodo%2Fget_items%3Freturn_response%3Dtrue%22%2C%22authentication%22%3A%22genericCredentialType%22%2C%22genericAuthType%22%3A%22httpHeaderAuth%22%2C%22sendHeaders%22%3Atrue%2C%22headerParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22content-type%22%2C%22value%22%3A%22application%2Fjson%22%7D%5D%7D%2C%22sendBody%22%3Atrue%2C%22specifyBody%22%3A%22json%22%2C%22jsonBody%22%3A%22%7B%20%5Cn%20%20%5C%22entity_id%5C%22%3A%20%5C%22todo.spesa%5C%22%2C%5Cn%20%20%5C%22status%5C%22%3A%20%5C%22needs_action%5C%22%5Cn%7D%5Cn%22%2C%22options%22%3A%7B%22allowUnauthorizedCerts%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B-300%2C900%5D%2C%22id%22%3A%22fd3e1744-1602-4092-b137-becf9b1f0f5e%22%2C%22name%22%3A%22Get%20current%20items%22%2C%22alwaysOutputData%22%3Afalse%2C%22credentials%22%3A%7B%22httpHeaderAuth%22%3A%7B%22id%22%3A%2238GE3o8R9bF7oiro%22%2C%22name%22%3A%22Header%20Auth%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22method%22%3A%22POST%22%2C%22url%22%3A%22https%3A%2F%2F%5BHome_Assistant_IP%5D%3A%5Bport%5D%2Fapi%2Fservices%2Ftodo%2Fadd_item%22%2C%22authentication%22%3A%22genericCredentialType%22%2C%22genericAuthType%22%3A%22httpHeaderAuth%22%2C%22sendHeaders%22%3Atrue%2C%22headerParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22content-type%22%2C%22value%22%3A%22application%2Fjson%22%7D%5D%7D%2C%22sendBody%22%3Atrue%2C%22specifyBody%22%3A%22json%22%2C%22jsonBody%22%3A%22%3D%7B%20%5Cn%20%20%5C%22entity_id%5C%22%3A%20%5C%22todo.spesa%5C%22%2C%5Cn%20%20%5C%22item%5C%22%3A%20%5C%22%7B%7B%20%24json.ingredient%20%7D%7D%5C%22%2C%5Cn%20%20%5C%22description%5C%22%3A%20%5C%22%7B%7B%20%24json.quantity%20%7D%7D%5C%22%5Cn%7D%22%2C%22options%22%3A%7B%22allowUnauthorizedCerts%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B1720%2C680%5D%2C%22id%22%3A%22dcb3920a-7c90-4391-b85e-d7d2f33179ea%22%2C%22name%22%3A%22Add%20new%20items%20to%20list%22%2C%22alwaysOutputData%22%3Afalse%2C%22credentials%22%3A%7B%22httpHeaderAuth%22%3A%7B%22id%22%3A%2238GE3o8R9bF7oiro%22%2C%22name%22%3A%22Header%20Auth%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22mode%22%3A%22combine%22%2C%22advanced%22%3Atrue%2C%22mergeByFields%22%3A%7B%22values%22%3A%5B%7B%22field1%22%3A%22ingredient%22%2C%22field2%22%3A%22summary%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.merge%22%2C%22typeVersion%22%3A3%2C%22position%22%3A%5B1180%2C440%5D%2C%22id%22%3A%22481b32ca-1416-4b37-b5bb-9bae4d0c36ca%22%2C%22name%22%3A%22Merge%20actual%20items%22%7D%2C%7B%22parameters%22%3A%7B%22modelId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gpt-3.5-turbo%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22GPT-3.5-TURBO%22%7D%2C%22messages%22%3A%7B%22values%22%3A%5B%7B%22content%22%3A%22%3Dfind%20if%20there%20are%20some%20quantity%20in%20the%20text%20%5C%22%7B%7B%20%24json.quantity%20%7D%7D%5C%22%20and%20%5C%22%7B%7B%20%24json.description%20%7D%7D%5C%22%20and%20in%20case%20sum%20them%20in%20the%20field%20NewDescription%5C%22%3B%20return%20a%20json%20object%20with%20%5C%22Ingredient%5C%22%20%3D%20%5C%22%20%7B%7B%20%24json.ingredient%20%7D%7D%20and%20%20%5C%22NewDescription%5C%22%22%7D%5D%7D%2C%22jsonOutput%22%3Atrue%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.openAi%22%2C%22typeVersion%22%3A1.7%2C%22position%22%3A%5B1360%2C440%5D%2C%22id%22%3A%227841ba7a-4190-4005-95d9-2de77e93b167%22%2C%22name%22%3A%22Update%20quantity%22%2C%22credentials%22%3A%7B%22openAiApi%22%3A%7B%22id%22%3A%2269QZlxxBRyU7gp8p%22%2C%22name%22%3A%22OpenAi%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22method%22%3A%22POST%22%2C%22url%22%3A%22https%3A%2F%2F%5BHome_Assistant_IP%5D%3A%5Bport%5D%2Fapi%2Fservices%2Ftodo%2Fupdate_item%22%2C%22authentication%22%3A%22genericCredentialType%22%2C%22genericAuthType%22%3A%22httpHeaderAuth%22%2C%22sendHeaders%22%3Atrue%2C%22headerParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22content-type%22%2C%22value%22%3A%22application%2Fjson%22%7D%5D%7D%2C%22sendBody%22%3Atrue%2C%22specifyBody%22%3A%22json%22%2C%22jsonBody%22%3A%22%3D%7B%20%5Cn%20%20%5C%22entity_id%5C%22%3A%20%5C%22todo.spesa%5C%22%2C%5Cn%20%20%5C%22item%5C%22%3A%20%5C%22%7B%7B%20%24json.message.content.Ingredient%20%7D%7D%5C%22%2C%5Cn%20%20%5C%22description%5C%22%3A%20%5C%22%7B%7B%20%24json.message.content.NewDescription%20%7D%7D%5C%22%5Cn%7D%22%2C%22options%22%3A%7B%22allowUnauthorizedCerts%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B1720%2C440%5D%2C%22id%22%3A%228ee80a2b-1e8d-4114-9ed9-bff8b973991b%22%2C%22name%22%3A%22Publish%20update%20to%20actual%20items%22%2C%22alwaysOutputData%22%3Afalse%2C%22credentials%22%3A%7B%22httpHeaderAuth%22%3A%7B%22id%22%3A%2238GE3o8R9bF7oiro%22%2C%22name%22%3A%22Header%20Auth%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22updates%22%3A%5B%22message%22%5D%2C%22additionalFields%22%3A%7B%22download%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.telegramTrigger%22%2C%22typeVersion%22%3A1.1%2C%22position%22%3A%5B-840%2C580%5D%2C%22id%22%3A%226d386f16-fdd6-4027-817d-65cf1c4f530b%22%2C%22name%22%3A%22Telegram%20Trigger%22%2C%22webhookId%22%3A%223af2ef6c-35e3-483e-a69c-6bc3bb777126%22%2C%22credentials%22%3A%7B%22telegramApi%22%3A%7B%22id%22%3A%22iXsm1Y2bgQiCjsTf%22%2C%22name%22%3A%22Telegram%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22rules%22%3A%7B%22values%22%3A%5B%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22loose%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.link_preview_options%20%7D%7D%22%2C%22rightValue%22%3A%22%22%2C%22operator%22%3A%7B%22type%22%3A%22string%22%2C%22operation%22%3A%22exists%22%2C%22singleValue%22%3Atrue%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22renameOutput%22%3Atrue%2C%22outputKey%22%3A%22link%22%7D%2C%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22loose%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%22e249e94e-382f-43ec-b696-f764158a6d39%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.photo%20%7D%7D%22%2C%22rightValue%22%3A%22%22%2C%22operator%22%3A%7B%22type%22%3A%22array%22%2C%22operation%22%3A%22exists%22%2C%22singleValue%22%3Atrue%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22renameOutput%22%3Atrue%2C%22outputKey%22%3A%22img%22%7D%2C%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22loose%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%2236863371-60cf-4357-8eb6-7337f7f01e0a%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.text%20%7D%7D%22%2C%22rightValue%22%3A%22%22%2C%22operator%22%3A%7B%22type%22%3A%22string%22%2C%22operation%22%3A%22notEmpty%22%2C%22singleValue%22%3Atrue%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22renameOutput%22%3Atrue%2C%22outputKey%22%3A%22text%22%7D%2C%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22loose%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%22e45fc880-e8fb-4b4f-a6db-8fb86a570915%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.document.mime_type%20%7D%7D%22%2C%22rightValue%22%3A%22application%2Fpdf%22%2C%22operator%22%3A%7B%22type%22%3A%22string%22%2C%22operation%22%3A%22equals%22%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22renameOutput%22%3Atrue%2C%22outputKey%22%3A%22pdf%22%7D%5D%7D%2C%22looseTypeValidation%22%3Atrue%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.switch%22%2C%22typeVersion%22%3A3.2%2C%22position%22%3A%5B-280%2C220%5D%2C%22id%22%3A%22dfad7030-03f4-47d2-a6b6-73adbc966cd3%22%2C%22name%22%3A%22Get%20content%20type%22%7D%2C%7B%22parameters%22%3A%7B%22url%22%3A%22%3D%7B%7B%20%24json.message.link_preview_options.url%20%7D%7D%22%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B0%2C0%5D%2C%22id%22%3A%2296b77c37-8120-4fbd-937a-8b69b4bc65d2%22%2C%22name%22%3A%22Open%20link%22%7D%2C%7B%22parameters%22%3A%7B%22promptType%22%3A%22define%22%2C%22text%22%3A%22Extract%20the%20ingredients%20of%20the%20recipe%20end%20return%20them%20in%20a%20json%22%2C%22hasOutputParser%22%3Atrue%2C%22messages%22%3A%7B%22messageValues%22%3A%5B%7B%22type%22%3A%22HumanMessagePromptTemplate%22%2C%22message%22%3A%22%3D%7B%7B%20%24json.data%20%7D%7D%22%7D%5D%7D%7D%2C%22id%22%3A%22faad564b-6cd0-489e-a104-0b938ad33359%22%2C%22name%22%3A%22Extract%20ingredients%20from%20link%22%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.chainLlm%22%2C%22typeVersion%22%3A1.4%2C%22position%22%3A%5B240%2C0%5D%7D%2C%7B%22parameters%22%3A%7B%22promptType%22%3A%22define%22%2C%22text%22%3A%22Extract%20the%20ingredients%20of%20the%20recipe%20end%20return%20them%20in%20a%20json%22%2C%22hasOutputParser%22%3Atrue%2C%22messages%22%3A%7B%22messageValues%22%3A%5B%7B%22type%22%3A%22HumanMessagePromptTemplate%22%2C%22messageType%22%3A%22imageBinary%22%7D%5D%7D%7D%2C%22id%22%3A%22bfd3cc8d-d943-4672-b6cb-36519b15aa2f%22%2C%22name%22%3A%22Extract%20ingredients%20from%20image%22%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.chainLlm%22%2C%22typeVersion%22%3A1.4%2C%22position%22%3A%5B240%2C220%5D%7D%2C%7B%22parameters%22%3A%7B%22promptType%22%3A%22define%22%2C%22text%22%3A%22Extract%20the%20ingredients%20of%20the%20recipe%20end%20return%20them%20in%20a%20json%3B%20if%20it%20doesn%27t%20look%20like%20a%20recepite%20but%20a%20single%20ingredient%2C%20treat%20is%20as%20an%20such%3B%20if%20no%20quantity%20is%20specified%2C%20omit%20it%22%2C%22hasOutputParser%22%3Atrue%2C%22messages%22%3A%7B%22messageValues%22%3A%5B%7B%22type%22%3A%22HumanMessagePromptTemplate%22%2C%22message%22%3A%22%3D%7B%7B%20%24json.text%20%7D%7D%22%7D%5D%7D%7D%2C%22id%22%3A%2294e3e7aa-42c9-43c2-88f6-e81e19dd44df%22%2C%22name%22%3A%22Extract%20ingredients%20from%20text%22%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.chainLlm%22%2C%22typeVersion%22%3A1.4%2C%22position%22%3A%5B240%2C480%5D%7D%2C%7B%22parameters%22%3A%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22loose%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%2211fcd2ba-3e41-4771-ad58-3b46f305a3e8%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.chat.id%20%7D%7D%22%2C%22rightValue%22%3A%22%5BChat_ID%5D%22%2C%22operator%22%3A%7B%22type%22%3A%22number%22%2C%22operation%22%3A%22equals%22%7D%7D%2C%7B%22id%22%3A%22cbfe3a40-b005-4543-87c4-9888e74b7fd9%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.message.from.id%20%7D%7D%22%2C%22rightValue%22%3A%22%5BUser_ID%5D%22%2C%22operator%22%3A%7B%22type%22%3A%22number%22%2C%22operation%22%3A%22equals%22%7D%7D%5D%2C%22combinator%22%3A%22or%22%7D%2C%22looseTypeValidation%22%3Atrue%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.if%22%2C%22typeVersion%22%3A2.2%2C%22position%22%3A%5B-620%2C580%5D%2C%22id%22%3A%22edb6f524-24f1-4a43-80bb-25944faca6b3%22%2C%22name%22%3A%22Check%20authorized%20users%22%7D%2C%7B%22parameters%22%3A%7B%22resource%22%3A%22chat%22%2C%22operation%22%3A%22leave%22%2C%22chatId%22%3A%22%3D%7B%7B%20%24json.result.chat.id%20%7D%7D%22%7D%2C%22type%22%3A%22n8n-nodes-base.telegram%22%2C%22typeVersion%22%3A1.2%2C%22position%22%3A%5B0%2C1140%5D%2C%22id%22%3A%22818ba4c7-71c7-45ff-8406-8ddefa79a87c%22%2C%22name%22%3A%22Leave%20chat%22%2C%22credentials%22%3A%7B%22telegramApi%22%3A%7B%22id%22%3A%22iXsm1Y2bgQiCjsTf%22%2C%22name%22%3A%22Telegram%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22chatId%22%3A%22%3D%7B%7B%20%24json.message.chat.id%20%7D%7D%22%2C%22text%22%3A%22You%27re%20not%20authorized%20to%20use%20this%20bot%22%2C%22replyMarkup%22%3A%22replyKeyboardRemove%22%2C%22replyKeyboardRemove%22%3A%7B%7D%2C%22additionalFields%22%3A%7B%22appendAttribution%22%3Afalse%7D%7D%2C%22type%22%3A%22n8n-nodes-base.telegram%22%2C%22typeVersion%22%3A1.2%2C%22position%22%3A%5B-300%2C1140%5D%2C%22id%22%3A%2269b03a74-9d11-4d1f-8be2-8813a6458f92%22%2C%22name%22%3A%22Unauthorized%20reply%22%2C%22credentials%22%3A%7B%22telegramApi%22%3A%7B%22id%22%3A%22iXsm1Y2bgQiCjsTf%22%2C%22name%22%3A%22Telegram%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22assignments%22%3A%7B%22assignments%22%3A%5B%7B%22id%22%3A%225b7ac878-ad98-49a7-8095-ca4a15ff2c31%22%2C%22name%22%3A%22text%22%2C%22value%22%3A%22%3D%7B%7B%20%24json.message.text%20%7D%7D%22%2C%22type%22%3A%22string%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.set%22%2C%22typeVersion%22%3A3.4%2C%22position%22%3A%5B0%2C380%5D%2C%22id%22%3A%22d95f5106-3bcf-409d-a59f-5ec18266dac9%22%2C%22name%22%3A%22Get%20message%20text%22%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22pdf%22%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.extractFromFile%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B0%2C580%5D%2C%22id%22%3A%22edf92630-9fbf-4811-9e2e-6f72d213ebdb%22%2C%22name%22%3A%22Get%20PDF%20Content%22%7D%5D%2C%22connections%22%3A%7B%22OpenAI%20Chat%20Model%22%3A%7B%22ai_languageModel%22%3A%5B%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20image%22%2C%22type%22%3A%22ai_languageModel%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Extract%20ingredients%20from%20text%22%2C%22type%22%3A%22ai_languageModel%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Extract%20ingredients%20from%20link%22%2C%22type%22%3A%22ai_languageModel%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Structured%20Output%20Parser1%22%3A%7B%22ai_outputParser%22%3A%5B%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20image%22%2C%22type%22%3A%22ai_outputParser%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Extract%20ingredients%20from%20text%22%2C%22type%22%3A%22ai_outputParser%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Extract%20ingredients%20from%20link%22%2C%22type%22%3A%22ai_outputParser%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20new%20items%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Add%20new%20items%20to%20list%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22List%20current%20items%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Merge%20actual%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A1%7D%2C%7B%22node%22%3A%22Get%20new%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A1%7D%5D%5D%7D%2C%22List%20new%20items%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Merge%20actual%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Get%20new%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20current%20items%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22List%20current%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Merge%20actual%20items%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Update%20quantity%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Update%20quantity%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Publish%20update%20to%20actual%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Telegram%20Trigger%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Check%20authorized%20users%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20content%20type%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Open%20link%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20image%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Get%20message%20text%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Get%20PDF%20Content%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Open%20link%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20link%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Extract%20ingredients%20from%20link%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22List%20new%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Extract%20ingredients%20from%20image%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22List%20new%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Extract%20ingredients%20from%20text%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22List%20new%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Check%20authorized%20users%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Get%20content%20type%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%2C%7B%22node%22%3A%22Get%20current%20items%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Unauthorized%20reply%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Unauthorized%20reply%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Leave%20chat%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20message%20text%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20text%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20PDF%20Content%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Extract%20ingredients%20from%20text%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%7D%2C%22pinData%22%3A%7B%7D%7D' frame=true></n8n-demo>

<br/>



### **Workflow Description**

1. **Trigger**:
- A **Telegram Trigger** monitors all the interactions with the bot.
- Configure it using the `[Telegram_Token]` retrieved at Step 1.
- With this trigger you cannot debug the workflow while active, so you need to temporary deactivate it in you want to manually use the **Test Workflow** or **Test Node** button.

2. **Authorization Check**:
- Verifies that the user is authorized to interact with the bot, since Telegram bot created with BotFather are public
- I checked both the ID of the Telegram group used by my wife and me and also mi user ID, if I want to chat directly with the bot for debugging purposes. You can retrieve the ID's by debugging the node with a simple test call

3. **Unauthorized reply**, **Leave chat**:
- Non authorized users/groups receive a *"You're not authorized to use this bot"* message.
- If the bot was added in a group, automatically leaves it.
- If you don't uncheck **Append n8n Attribution**, it will append the phrase *"This message was sent automatically with n8n”* to the end of the message
  
4. **Get Content Type**:
- Depending on the content type (text, link, image, PDF), routes the input for further processing

5. **Content Normalization**:
- **Text:** Transform the output structure in order to reuse tha same node as PDF (**Get message text**).
- **Link:** Opens the URL (**Open link**).
- **Image:** No conversion needed.
- **PDF:** Get PDF content in a JSON format.

6. **Content processing**:
- Three **Basic LLM Chain** nodes process the message content.
- LLM Chain nodes that process **Link** and **Image** uses a simple prompt: `Extract the ingredients of the recipe end return them in a json`
- For the **Text**, I had to provide some additional information; otherwise, the AI might have processed the content differently. Specifically, I told it to treat as ingredients any words that could directly represent one (otherwise, instead of "chips," it might have returned the recipe for French fries) and to omit the quantity if not specified (to avoid ending up with "n/a"). Here is the prompt used:
`Extract the ingredients of the recipe end return them in a json; if it doesn't look like a recepite but a single ingredient, treat is as an such; if no quantity is specified, omit it`
- Each node shares the same **OpenAI Chat Model** (**GPT-4O-MINI**) and the same **Structured Output Parser**, in order to return data always with the same structure, which is so defined:
  
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "ingredient": { "type": "string" },
      "quantity": { "type": "string" }
    }
  }
}
```

7. **Get current items**
- Fetches items from Home Assistant's shopping list (`todo.spesa`) via API, by calling the `https://[Home_Assistant_IP]:[port]/api/services/todo/get_items?return_response=true`
- `return_response=true` is mandatory in order to receive the structured output
- Replace the `[Home_Assistant_IP]`, `[port]` and adjust protocol (`http`/`https`) accordingly
- Create a new **Header Auth** account by using the `Long-Lived Token` created at Step 2.
- Toggle **Send Headers** and add the Header parameter `content-type`=`application/json`
- Toggle **Send Body** and specify a `JSON` Body like this, replacing `todo.spesa` with the name of your ToDo list.
```json
{ 
  "entity_id": "todo.spesa",
  "status": "needs_action"
}
```
- `"status": "needs_action"` tells the API to skip the completed items and return only the active ones.

8. **List new items**, **List current items**
- Converts both newly parsed items and existing items from the shopping list into individual entries for comparison

9. **Data Comparison**
- **Get new items** keeps only the non-matched items from the previous lists: these are the ingredients to add to our shopping list
- **Merge actual items** keeps matched items: these are the ingredients already present in list, but me may need to update quantity

10. **Add new items to list**
- Calls the `https://[Home_Assistant_IP]:[port]/api/services/todo/add_item` for each element of the **Get new items** list
- Replace the `[Home_Assistant_IP]`, `[port]` and adjust protocol (`http`/`https`) accordingly
- Use the same **Authorization header**, **Headers Parameters** and **Body Content Type** as before
- Use a JSON like this for the body, replacing `todo.spesa` with the name of your ToDo list.
```json
{ 
  "entity_id": "todo.spesa",
  "item": "{{ $json.ingredient }}",
  "description": "{{ $json.quantity }}"
}
```

11. **Update quantity**
- Since I don’t know how the quantity is specified (formatting, units of measurement, ...) either in the item already on the list or in the one obtained from the recipe, I let the AI interpret the data, normalize it, add the new quantity accordingly and return in a structured format.
- This is the prompt I use with **GPT-3.5-TURBO** model:
`find if there are some quantity in the text "{{ $json.quantity }}" and "{{ $json.description }}" and in case sum them in the field NewDescription"; return a json object with "Ingredient" = " {{ $json.ingredient }} and  "NewDescription"`

12. **Publish update to actual items**
- Calls the `https://[Home_Assistant_IP]:[port]/api/services/todo/update_item` for each element of the **Merge actual items** list
- Replace the `[Home_Assistant_IP]`, `[port]` and adjust protocol (`http`/`https`) accordingly
- Use the same **Authorization header**, **Headers Parameters** and **Body Content Type** as before
- Use a JSON like this for the body, replacing `todo.spesa` with the name of your ToDo list.
```json
{ 
  "entity_id": "todo.spesa",
  "item": "{{ $json.message.content.Ingredient }}",
  "description": "{{ $json.message.content.NewDescription }}"
}
```

<br/><br/>

# Step 5: Chat with your bot
You can directly open a Telegram chat with your bot or add it in a group and promote it to **Administrator**. In this case remember to update the `Group ID` in the **Check authorized users** node in the workflow.

<br/><br/>

# Step 6: Enjoy
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