# AI Bill Assistant

### (aka why spend time for downloading and organizing recurring bill when an AI can do It for you?)

[![View on SmartHomeTricks](https://img.shields.io/badge/View_on-SmartHomeTricks-blue?style=for-the-badge&logo=homeassistant&logoColor=white)](https://smarthometricks.sirri.it/use_cases/ai-bill-assistant/)

Forget about spending time to save and organize your bills, an AI based workflow will do it for you! This workflow read your bill related email, identify where the PDF bill is stored (it doesn't matter if attached to the email, linked in the email body or in a personal area website), extract main info (like the amount, reference period, etc)  and save the PDF and also structured bill's data in a shared folder.

<sub>***Date*:** *25/12/2024*<br/>***Tag:*** *Open AI, n8n, REST API*</sub>

---

![preview](./ai-bill-assistant.webp)

I hate recurring manual tasks... I consider them a waste of time, and whenever possible, I try to automate them as much as I can; that way, I can invest that time in something more productive and interesting. One of these tasks is undoubtedly filing the various utility bills each month: I always use direct debit and opt for digital billing, which makes the environment happier by saving paper and saves me from having to manually scan paper documents. The problem is that every month, when I receive the email notification about a new bill, I have to:
- open the email
- find where is the bill, since it depends on the cases:
  - if the bill is attached, it's easy: just save it
  - sometimes there is a link to the bill instead,so in that case, I just need to find it by reading the email and save the PDF
  - other times, however, there is a link to the reserved area, so I have to navigate the site, identify the bill, and save it from there
- in any case, the file then needs to be renamed according to my preferred naming convention (usually using the issue date or reference period to make it more readable than just the document number) and moved to the specific folder on the cloud.

With this trick, I have completely automated the entire process… so I have the emails neatly organized in their folder whenever I need them. Additionally, if necessary, the main information is automatically extracted, allowing it to be saved in an Excel sheet or used in another process/software/tool that requires raw data for monitoring consumption and/or expenses. 




# Step 1: Organize e-mails
This is an optional step but I recommend following it to simplify the process later.
I use Gmail as my main email address, so I simply defined different labels to mark all the bills I want to manage. Below you can find the instructions to configure Gmail as I did but the process may differ for other e-mail providers.

1. **Create a label for each service provider**
You can do it directly from the left-hand sidebar or by the **all settings** menu by clicking the gear icon in the top-right corner of Gmail web interface. If you want to nest this label under an existing one, check the box **"Nest label under"** and choose the parent label.
This is the label structure for my service providers:
![Label structure](label-structure.webp)

2. **Define a filter for each label**:
   - Click on the gear icon in the top-right corner of Gmail and select **"See all settings"**.
   - Go to the **"Filters and Blocked Addresses"** tab.
   - Click **"Create a new filter"**.
   - A dialog box will appear where you can define the criteria for the filter, for example filtering by sender email address or domain, keywords in the subject line, specific words in the email body, and so on...
   - Choose the best filter for your specific needs: it may vary between two different providers, but usually sender and/or subject does the job well.
   - Choose what happens to emails matching the criteria: in this case we want to apply the label just defined
   - Apply the filter also to existing emails, by checking the box **"Also apply filter to matching conversations"**.
![Label filter](label-filter.webp)

3. **Check the filters**
If everything worked, you will find all the emails with invoices under the label corresponding to each supplier. If not, check the filter configuration.  
Make sure the emails to be processed are marked as "unread," as this will be necessary for setting up the next step.




# Step 2: Set-Up Open AI API
We will use a **GPT LLM** for parsing e-mails, personal area websites and also PDFs. In this way, we are not tied to the format of the email/website/pdf, and if it changes in the future, we would only need to make minimal adjustments, if any, since the AI will always interpret the text to identify the parts we are interested in, based on our instructions.

If you don't already have an OpenAI API key, read the accordion below.

<details>
<summary>Set up the OpenAI API</summary>

We will use a **GPT LLM** to process unstructured data. You can choose other AI LLMs, but regardless of which one you use, to call it from another piece of software (like n8n) you need an authorised API key. Here are the steps to create one with OpenAI:

### 1. Create an OpenAI Account
- **Sign Up or Log In**: If you don't already have an OpenAI account, go to [OpenAI's website](https://platform.openai.com/signup) and sign up. If you already have an account, simply log in at [OpenAI Login](https://platform.openai.com/login).

### 2. Access the API Dashboard
- **Go to the API section**: After logging in, navigate to the OpenAI API dashboard. You can find this by going to [OpenAI Platform Dashboard](https://platform.openai.com/account/api-keys).

### 3. Generate an API Key
- **Create a New API Key**:
   - Once you're in the API keys section, click on **"Create new secret key"** or **"Create new API key"**.
   - OpenAI will generate a new API key for you. This key is the API token you'll use to authenticate requests to the OpenAI API.
- **Copy the Key**: After the key is generated, **copy it immediately** because it will not be shown again for security reasons.

### 4. Store the API Key Securely
- **Secure Storage**: Store the API key in a safe place, like a password manager.

### 5. Monitor Usage and Billing
1. **Monitor API Usage**: OpenAI provides detailed usage analytics on your dashboard. You can monitor the number of tokens you've used and manage your spending.
2. **Manage Quotas**: If needed, you can set limits or alerts for your API usage to avoid unexpected charges.

</details>





# Step 3: Create the automation flow
For this process, I used **n8n**. If you don't already have n8n installed, read the accordion below.

<details>
<summary>Set up n8n</summary>

**n8n** is an open-source workflow automation tool that allows you to connect various applications and services to automate repetitive tasks without manual intervention. It provides a visual interface where you can design workflows by linking different nodes that represent actions, triggers, or data processing steps. 

I installed it in a Proxmox LXC by using the helper script provided by [Community-Scripts](https://community-scripts.github.io/ProxmoxVE): this is a very useful Community with many script that will help you many times: if you like them, consider donating to support Angie, tteckster's wife - the founder and best supporter of the community - too early passed away.

</details>


I created three different workflows for three different suppliers; each one uses a unique approach and applies different solutions, based on my needs as well as to provide various ideas for study and exploration.  

Of course, your needs might differ, as well as the structure of the emails from your suppliers. However, you can draw inspiration from the three workflows and the solutions implemented to build one that best suits your requirements and adapts to your specific case.

1. **Bill included as an email attachment**: the simpliest case ☺️: the attachment will be renamed and saved to a shared folder. But to make it more interesting, it processes all unread email and extracts and organizes relevant details (e.g., bill number, dates, and phone number) from the attached PDF.
2. **Bill downloadable via a link contained in the email body**: in this case we don't need to retrieve additional information from the PDF. The attachment will be renamed and saved to a shared folder; the difference from the previous case lies in the method of retrieving the PDF, by parsing the email content to extract the link of the PDF. Unlike the other workflows, this one processes just a single email, since the link provided has an expiration date and for older emails it doesn't work.
3. **Bill downloadable via a link from the personal area, which is accessible through a link contained in the email body** : this is the most complete case that groups all the solutions used by the previous ones, including the extraction of key information, which this time will be also saved in a JSON file for further applications. 




## **Workflow description: nodes common to each solution**

- **Trigger**
   - In addition to the webhook, there is a scheduled trigger to process new emails and bills automatically on the last days of every month.

- **Fetch Unread Emails**
   - Connects to a Gmail account and retrieves unread emails tagged with the label defined during the first step.
   - Uncheck **Simplify** in order to retrieve all the raw data
   - Uncheck **Downloads attachments**, except the first case when the bill is effectively attached to the email

- **Loop Over Items**
   - Iterates over each email retrieved, processing them individually.
   - At the end give response to webhook with the result of the loop (you can improve this part with a more detailed json output and error management)

- **Convert PDF to JSON**
   - Downloads and parses the PDF content into JSON format.
   - Prepares the extracted text for further analysis, such as date and bill information.

- **Create subfolder**
    - Creates a subfolder on a NAS based on the supplier and emission year (e.g., `/nas/<supplier_name>/2024/`).
    ``` 
    mkdir -p /nas/<supplier_name>/{{ $json.message.content.emission_date.substring(0,4) }}
    ```
    - `<supplier_name>` will change for each case
    - The main path `/nas` is a remote folder previously mounted in the n8n server with one subfolder for each supplier already created

- **Mark Email as Read**
   - At the end it marks the processed email as read to prevent reprocessing in subsequent runs, keeping the workflow efficient.




## 1. Bill included as an email attachment

![n8n workflow: Salva bollette Wind](./ai-bill-assistant-n8n-1.webp)

> **n8n Workflow: Salva bollette Wind**
>
> [Download workflow JSON](./ai-bill-assistant-n8n-1.json) and import it into your n8n instance.





This is the automation I use with **Wind** mobile and internet provider; the flow:
1. Processes unread emails containing Wind bills.
2. Extracts and organizes relevant details (e.g., bill number, dates, and phone number) from the attached PDF.
3. Saves the processed PDF to a network drive (NAS) with a structured naming convention and directory organization.
4. Marks the email as read to avoid reprocessing.




### **Workflow description**
In addition to the common part:

- **Fetch Unread Emails (`Get unread Wind messages`)**
   - Remember to check **Downloads attachments** this time

- **Extract Bill Details (`Retrieve info`)**
   - Uses GPT-3.5 to intelligently extract structured data from unstructured text in the PDF and retrieve key information:
     - Telephone number (`tel_num`).
     - Bill number (`bill_number`).
     - Emission date (`emission_date`).
     - Reference period (`start_date` and `end_date`).
   - Ensures all dates are in ISO format and places all extracted properties at the root level of the JSON.
  
    Here is the prompt I used; I specified the name of the fields and the date format in order to have an output consistent and equal between each run:
    ```
    extract from {{ $json.text }} the telephone number (tel_num), the bill number (bill_number), the emission date (emission_date) and the reference period (start_date, end_date). 
    All the dates in ISO format and all the properties must be at root level.
    ```

- **Organize and Save PDF (`Save PDF to NAS share`)**
   - Saves the PDF with a structured file name, including:
     - Emission date
     - Bill number
     - Telephone number
   - Example file path:
     ```
     /nas/Wind/2023/2023-12-24 - 123456789 - 1234567890.pdf
     ```




This is how appears the NAS folder with all the bills extracted and organized by this flow. In my case this is not the target destination but a scheduled task move the bills from the network share to mi iCloud document folder.
![folder structure wind](folder-structure-wind.webp)




## 2. Bill downloadable via a link contained in the email body

![n8n workflow: Salva bollette Acegas](./ai-bill-assistant-n8n-2.webp)

> **n8n Workflow: Salva bollette Acegas**
>
> [Download workflow JSON](./ai-bill-assistant-n8n-2.json) and import it into your n8n instance.






This is the automation I use with **Acegas**, local water supplier; the flow:
1. Processes last unread emails containing Acegas bills. As I said before, the link to download the bill has an expiration date: it is no longer accessible once the next bill becomes available, so reading the latest one is sufficient.
2. Doesn't extract any data but it still uses AI to intelligently parse email content and locate the precise link for downloading the bill.
3. Saves the processed PDF to a network drive (NAS) with a structured naming convention and directory organization.
4. Marks the email as read to avoid reprocessing.




### **Workflow description**
In addition to the common part:

- **Fetch Unread Emails (`Gmail`)**
   - Here the difference is that it limits the retrieval to one email at a time to process each bill individually.
   - You can skip **Downloads attachments** this time

- **Extract 'Area Riservata' Link (`Retrieve 'area riservata' link`)**
   - Uses OpenAI GPT-3.5 to parse the email's HTML content and extract the link labeled "Visualizza il documento."
   - Ensures that only the relevant PDF download link is returned for further processing.

      Here is the prompt I used, really really simple:
    ```
    select the "Visualizza il documento" link in  {{ $json.html }} and return just the link 
    ```

- **Download HTML Page (`Get html page`)**
   - Accesses the extracted link to retrieve the HTML content that contains the PDF document.

-  **Format Date (`Date & Time`)**
   - Extracts the creation date from the JSON content (using metadata like `CreationDate`).
   - Converts the date into a readable format (`yyyy-MM-dd`) for naming and organizing the files.

- **Merge Metadata with PDF (`Merge Info with PDF`)**
   - Combines the processed information (e.g., formatted date) with the PDF file data for structured handling.

- **Save PDF to NAS (`Save PDF to NAS share`)**
   - Renames and saves the PDF file in the created directory.
   - File naming convention includes the formatted date:
     ```
     /nas/Acegas/2023/2023-12-24.pdf
     ```


### **Margin Notes**
Thanks to AI, this automation is truly simple... without it, I would have had to analyze the document model of the email to identify the correct tag with the link... and redo everything if the format, layout, or structure of the email were to change.




## **3. Bill downloadable via a link from the personal area, which is accessible through a link contained in the email body**

![n8n workflow: Salva bollette ENI](./ai-bill-assistant-n8n-3.webp)

> **n8n Workflow: Salva bollette ENI**
>
> [Download workflow JSON](./ai-bill-assistant-n8n-3.json) and import it into your n8n instance.






This is the automation I use with **Eni**, a big Italian gas and electricity supplier; the flow:
1. Fetches unread email messages with Eni label.
2. Extracts a PDF link from the email content.
3. Downloads and processes the PDF to extract structured data.
4. Saves the processed PDF to a network drive (NAS) with organized filenames and directories.
5. Marks the processed email as read to avoid reprocessing.




### **Workflow description**
In addition to the common part:

- **Extract 'Area Riservata' Link (`Retrieve 'area riservata' link`)**
   - Uses OpenAI's GPT-3.5-Turbo to extract a specific link starting with `https://interattiva.eniplenitude.com` from the email content.

      Like the previous case, the prompt is really relly simple:
    ```
    select the link in  {{ $json.html }} that starts with "https://interattiva.eniplenitude.com" 
    ```

- **Retrieve HTML Page (`Get html page`)**
   - Downloads the HTML page linked in the email.

- **Extract PDF Link (`Retrive PDF link`)**
   - Analyzes the downloaded HTML using OpenAI GPT to locate the specific PDF link for the bill.
   - The prompt is, again, really simple but it uses `GPT-4O-MINI`, since the `3.5-TURBO`, used for all the other AI interactions, is not sufficient to elaborate all the data of the web page. 
    ```
    get pdf link from  {{ $json.data }}
    ```

- **Download and Process PDF (`Get PDF content` & `Convert PDF to JSON`)**
   - Downloads the PDF and converts its content into a JSON format for structured analysis.

- **Extract Detailed Bill Information (`Retrieve info`)**
   - Uses GPT models to parse the PDF content and extract key details:
     - Total amount (`total_amount`).
     - Reference period (`start_date` and `end_date`).
     - Costs and consumption for electricity and gas.
     - Bill number and emission date.
  
    Here is the prompt I used; Like in the first case I specified the name of the fields and the date format in order to have an output consistent and equal between each run. In this case :
    ```
    extract from {{ $json.text }} the total amount (total_amount), 
    the reference period (reference_period.start_date and reference_period.end_date), 
    the amount of the "Conteggio luce" (electricity_usage.total_cost and electricity_usage.consumption) and 
    "Conteggio gas" (gas_usage.total_cost and gas_usage.consumption), 
    the bill number (bill_number) and 
    the emission date in both natural (emission_date) and ISO format (emission_date_ISO) and year (year). 
    All the properties must be at root level.
    ```

-  **Save PDF and JSON to NAS (`Save PDF to NAS share`, `Save JSON to NAS share` and `Convert JSON to binary`)**
   - Organizes PDFs into directories on a NAS device, named by year and bill details, using a dynamic path like:
     ```
     /nas/Eni/{year}/{emission_date_ISO} {bill_number}.pdf
     ```
  - The same naming convention and path is also used for json file with all the structured data. This data can be used in another process/software/tool that requires raw data for monitoring consumption and/or expenses. Since the node require a binary data, a preemptive conversion of the json is needed (`Convert JSON to binary` step).
    Here is an example of the json generated (you can improve it by filtering also the message part):
    ```json
    [{
      "index":0,
      "message":{
        "role":"assistant",
        "content":{
          "total_amount":205.34,
          "reference_period":{
            "start_date":"2024-05-01",
            "end_date":"2024-05-31"
          },
          "electricity_usage":{
            "total_cost":181.49,
            "consumption":194
          },
          "gas_usage":{
            "total_cost":16.85,
            "consumption":6
          },
          "bill_number":"M246287450",
          "emission_date":"18.06.2024",
          "emission_date_ISO":"2024-06-18",
          "year":2024,
          "pdf_link":"https://interattiva.eniplenitude.com/55af7b8a6a5746ebb4da541e51d0e9b2"
        },
      "refusal":null},
      "logprobs":null,
      "finish_reason":"stop"
    }]
    ```




This is how appears the NAS folder with all the bills extracted and organized by this flow, with both PDF and JSON files.
![folder structure eni](folder-structure-eni.webp)





# Step 4: Think how to use all your saved time
You don't have to worry about bills anymore... forgot them! Your bank takes care of the payments, while AI saves your invoices, organizes them for you, and extracts the key information.
You have plenty of extra time to focus on what matters most or brings you satisfaction.




# Step 5: Possible additional extensions:  
1) Disable email notifications and instead configure a notification to receive structured data with amounts, consumption, and other relevant details.  
2) Leverage AI to extract additional important information from the bill, such as contract changes, price increases, or other details, and include them in the notifications."

And there's one more step: this workflow keeps extracting bills, but the structured data can do more than fill a folder. In the [RAG knowledge base article](../bills-documents-rag-knowledge-base/), I feed that same data into a private RAG system — the natural extension of this trick, where you query your bills instead of browsing them.




# Step 6: Enjoy
If this trick has been useful to you, keep scrolling down and consider supporting me by clicking that beautiful blue button! 😅

---

Even if I'll try to keep all this pages updated, products change over time, technologies evolve... so some use cases may no longer be necessary, some syntax may change, some technologies or products may no longer be available.

Remember to make a backup before modifying configuration files and consult the official documentation if any concept is unclear or unfamiliar.

*Use this guide under your own responsibility.*

If this pages have been helpful, you can

<a href="https://www.buymeacoffee.com/moreno.sirri" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

<sub>This work and all the contents of this website are licensed under a **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
You can distribute, remix, adapt, and build upon the material in any medium or format, <u>for noncommercial purposes only by giving credit to the creator</u>. Modified or adapted material must be licensed under identical terms.
You can find the full license terms [here](https://creativecommons.org/licenses/by-nc-sa/4.0/?ref=chooser-v1)</sub>
