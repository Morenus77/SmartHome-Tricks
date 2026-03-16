---
title: 'Automatic Instagram Posting'
published: true
date: '03-01-2025 16:30'
media_order: spreadsheet-structure.webp
taxonomy:
    tag: [Open AI, n8n, Facebook REST API, Instagram, Facebook App, Google Drive]
image_url: https://smarthometricks.sirri.it/user/pages/01.home/_use-cases/automatic-instagram-poster/auto-instag-post.webp
metadata:
  'og:title': Automatic Instagram Posting
  'og:type': article
  'og:image': https://smarthometricks.sirri.it/user/pages/01.home/_use-cases/automatic-instagram-poster/preview.webp
  'og:url': https://smarthometricks.sirri.it/home/_use-cases/automatic-instagram-poster
  description: Forget about spending time to manage your IG account (or Facebook page, Linkedin, ...): an AI based workflow will do it for you! This workflow takes the images you’ve edited and prepared for posting on Instagram, analyzes them, suggests the most suitable description and hashtags based on the content of each photo, and posts them on your behalf. You have just to review the description and define the day you want to have each one published.
---

Forget about spending time to manage your IG account (or Facebook page, Linkedin, ...): an AI based workflow will do it for you! This workflow takes the images you’ve edited and prepared for posting on Instagram, analyzes them, suggests the most suitable description and hashtags based on the content of each photo, and posts them on your behalf.
You have just to review the description and define the day you want to have each one published.<br/>
It uses `n8n`, `Open AI`, `Facebook REST API`, `Instagram`, `Facebook App`, `Google Drive` and just a very little `javascript` programming.

===

<script src="https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@2.0.0/webcomponents-loader.js"></script>
<script src="https://www.unpkg.com/lit@2.0.0-rc.2/polyfill-support.js"></script>
<script type="module" src="https://cdn.jsdelivr.net/npm/@n8n_io/n8n-demo-component/n8n-demo.bundled.js"></script>

{assets:inline_css}
n8n-demo {
	--n8n-workflow-min-height: 500px;
}
{/assets}

# Automatic Instagram Posting
### (aka: don't have time to manage your social accounts? Let the AI do it for you!)

Okay, this isn't exactly a topic related to home automation or home management... but it uses some of the technologies and products we've already seen for a different purpose and that's still related to how technology can simplify our daily lives and save us precious time. So, if you also have this need, here's an idea to save a good chunk of time that you can reinvest in other activities while simultaneously maintaining your presence on social media. 

Personally, I have no interest or aspiration to create viral content or to climb the ranks of Facebook, Instagram, TikTok, or other algorithmic platforms through constant posting. However, I do enjoy sharing my photos with friends and those who appreciate my way of seeing the world through the lens of a camera. The problem is: I have very little time to do it... and definitely not the consistency to post daily or at regular intervals. 

With this trick, **you can prepare more than 30 days' worth of content** - or even more - **in just a couple of minutes**, and let this automation handle the rest for you. In this example, I’ll use Instagram, but the same approach works for Facebook, LinkedIn, Slack, and many other social media or sharing tools, making the process easily adaptable to other use cases. 

Let’s see how!


## How It Works:  
- Edit, prepare and select the photos as usual and copy them into a predefined folder.  
- Automatically, a process processes the images and thanks to AI populates a spreadsheet with the suggested content for the post (caption, hashtags).  
- Access this spreadsheet to review the content, make any necessary edits to the description, hashtags, etc., and choose a publication date.  
- On the designated day, a second process automatically posts the photo with the defined description and updates the spreadsheet to mark that task completed.  
![workflow](auto-instag-post.webp)

<br/><br/>

# Step 1: Install and configure n8n
For this process, again, I used **n8n**: n8n is an open-source workflow automation tool that allows you to connect various applications and services to automate repetitive tasks without manual intervention. It provides a visual interface where you can design workflows by linking different nodes that represent actions, triggers, or data processing steps. 

I installed it in a Proxmox LXC by using the helper script provided by <a href="https://community-scripts.github.io/ProxmoxVE">Community-Scripts</a>: this is a very useful Community with many script that will help you many times: if you like them, consider donating to support Angie, tteckster's wife - the founder and best supporter of the community - too early passed away.

The installation and configuration are very simple; once completed, I mounted a network share from the NAS on the local path `/nas/Instagram`, which I will use as an exchange directory.

<br/><br/>

# Step 2: Prepare a support spreadsheet
I used a **Google Sheets** file to support the process. This allows me to easily access the content for review before it gets published.

The file, which I named `Instagram posts`, has the following structure:

![spreadsheet structure](spreadsheet-struct.webp)

## Description:
- **Date** : the planned date for that post; column type: `DateTime`, format: `dd/MM/yyyy`. You can set the format you prefer but you have to check the following part accordingly.
- **Filename**, **Description**, **Hashtag** : self-explanatory
- **Today**: I faced issues filtering by the Date field using the **Google Sheet API**  in the workflow below. After reading several forums, many suggested using a calculated field as a filter. To avoid creating a formula and pre-populating it for all rows in the column, I created a macro using **Apps Scripts** that automatically populates the `Today` column based on changes to the `Date` column:

```js
function onEdit(e) {
  var range = e.range;
  var sheet = range.getSheet();
  var row = range.getRow();
  if (range.getColumn() === 1) {
   //Today
    var formula = `=A${row} = TODAY()`;
    sheet.getRange(row, 5).setFormula(formula);
  }
  else if (range.getColumn() === 4) {
   //Hashtag count
    var formula = `=LEN(D${row})-LEN(SUBSTITUTE(D${row};"#";""))`;
    sheet.getRange(row, 7).setFormula(formula);
  }
}
``` 
- **Status**: this will be updated by the workflow responsible for making the post, setting it to `Done` along with the date and time of publication.
- **Hashtag count**: indicate the number of hashtags used to avoid exceeding the maximum limit of 30 (as of the time of writing) set by Instagram. As with the "Today" field, the formula is injected by the script above.


<br/><br/>

# Step 3: Setup and configure Instagram API
To automatically post on Instagram, it is necessary to use the **Facebook Graph API**, which must be configured by defining an **Application** and setting up its behavior and permissions. This step is specific to Instagram (and Facebook with slight modifications), but it is likely that similar steps will be required for other social platforms. Check the developer guides for specific social networks to learn how to configure the APIs for integrating custom apps.

## Prerequisite
Make sure you have a business account for Instagram; otherwise, log in to Instagram and select *Switch to a Business Account.*

## 1. Create a Facebook page
- If you don't already have a facebook page for this business, create one: It is not necessary to maintain, update, share or promote it... it only serves as a support for publishing on Instagram and no content will be published there. It should technically be optional, but I found that creating it makes the subsequent configuration easier and more straightforward.
- Link the page to your Instagram account: since the procedure may vary, I suggest searching on a search engine for *"link an Instagram account to a Facebook page"* and following the instructions in the official documentation, which will certainly be up-to-date.

## 2. Create a Facebook App
- Go to [https://developers.facebook.com/apps](https://developers.facebook.com/apps) and create a new app.
- When asked what your app will do, select "Other"
- Croose "Business" app type, since the publish functionality is not available to private customers. 
- Provide a name, contact email address, and select your business profile from the dropdown list.
- Complete the steps and enter your Facebook password when prompted.
  
## 3. Configure App
- Go to [https://app.freeprivacypolicy.com](https://app.freeprivacypolicy.com/) and create a privacy policy for your app
- Save it and let it readable from internet, so you can for example store in Google Drive and generate a public share link.
- Go to basic app settings and past the link in the privacy policy url, than save changes
- From the dashboard or from the side menu, under "Add product", choose **Facebook Login for Business** and follow the steps.
- Then add also **Instagram** product and follow the configuration steps: after logged in with your instagram account, you will then see your `[Instagram_Account_ID]`: take note of it since you will need to configure the publishing workflow.
- Take the app live using the toggle at the top of the page.
- In the app dashboard, under the "Tools" menu, select "GraphAPI Explorer"
- Select the right "Meta App" on the right, than "User Token"
- Under "Permissions" select `pages_show_list`, `business_management`, `instagram_basic`, `instagram_content_publish`, `instagram_manage_comments`, `instagram_manage_insights` (the last two shouldn't be necessary just for posting, but I got an error without them). You can add more permission If you plan to extend the functionalities of your app.

## 4. Get Page Access Token
- Now in the same page, click on "Generate Access Token": you will be asked to login and specify the FB page and linked Instagram account: that's why creating a page simplify the process... because you can also choose to give access to all current and future pages, but if no one has your IG account linked, the configuration would be more tricky.
- At the end copy the generated Access Token

## 5. Extend the lifespan of the Access Token
- In the app dashboard, under the "Tools" menu, select "Access Token Debugger"
- Paste the copied access token into the debugger field, then click on "Debug"
- Take a look at the screen: the token will expire in about one hour
- Click "Extend Token" than again on "Debug" button to extend the token for three months
- If you see the "Extend Token" button again, repeat these steps
- Else copy the last token, go back on "Graph Explorer", paste over the previous one and click again on "Generate"; go back to the debugger and redo the process: after the first debug you should see "Never" as expire date
- Copy this `[Access_Token]` that you will use later to configure the publishing workflow. 
  
<br/><br/>

# Step 4: Define the process of preparing the post

This **n8n workflow** automates the definition of the description and hashtags and prepare the posts. It uses AI to analyze the pictures in the shared folder to determine the best description and hashtag based on the defined prompt and populate the just defined spreadsheet.

Here's a step-by-step breakdown:

<n8n-demo workflow='%7B%22nodes%22%3A%5B%7B%22parameters%22%3A%7B%22triggerOn%22%3A%22folder%22%2C%22path%22%3A%22%2Fnas%2FInstagram%2F%22%2C%22events%22%3A%5B%22add%22%2C%22unlink%22%5D%2C%22options%22%3A%7B%22awaitWriteFinish%22%3Atrue%2C%22usePolling%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.localFileTrigger%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-2380%2C-2120%5D%2C%22id%22%3A%221530aa4d-1a1b-43e6-9288-8b939db0efc1%22%2C%22name%22%3A%22Local%20File%20Trigger%22%7D%2C%7B%22parameters%22%3A%7B%22amount%22%3A1%7D%2C%22id%22%3A%22244c6f96-0ba2-4e88-91ab-ea33c38a6697%22%2C%22name%22%3A%22Wait%201s%22%2C%22type%22%3A%22n8n-nodes-base.wait%22%2C%22typeVersion%22%3A1.1%2C%22position%22%3A%5B-2120%2C-2120%5D%2C%22webhookId%22%3A%2258bd81e9-d956-42cb-8d7d-62ed066b9b2d%22%7D%2C%7B%22parameters%22%3A%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22strict%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%220b60521a-ab63-4e9a-a0ca-19928eb99b99%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24(%27Local%20File%20Trigger%27).item.json.event%20%7D%7D%22%2C%22rightValue%22%3A%22add%22%2C%22operator%22%3A%7B%22type%22%3A%22string%22%2C%22operation%22%3A%22equals%22%2C%22name%22%3A%22filter.operator.equals%22%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.if%22%2C%22typeVersion%22%3A2.2%2C%22position%22%3A%5B-1680%2C-2120%5D%2C%22id%22%3A%225ead5436-0d6e-4168-9008-faaa71b2a176%22%2C%22name%22%3A%22Check%20if%20added%20or%20deleted%22%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22append%22%2C%22documentId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5Byour_document_id%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%20posts%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%3Fusp%3Ddrivesdk%22%7D%2C%22sheetName%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gid%3D0%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%23gid%3D0%22%7D%2C%22columns%22%3A%7B%22mappingMode%22%3A%22defineBelow%22%2C%22value%22%3A%7B%22Filename%22%3A%22%3D%7B%7B%20%24(%27Get%20filename%27).item.json.Filename%20%7D%7D%22%2C%22Description%22%3A%22%3D%7B%7B%20%24json.content%20%7D%7D%22%2C%22Hashtag%22%3A%22%3D%7B%7B%20%24json.hashtag%20%7D%7D%22%7D%2C%22matchingColumns%22%3A%5B%5D%2C%22schema%22%3A%5B%7B%22id%22%3A%22Date%22%2C%22displayName%22%3A%22Date%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%2C%7B%22id%22%3A%22Filename%22%2C%22displayName%22%3A%22Filename%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%7D%2C%7B%22id%22%3A%22Description%22%2C%22displayName%22%3A%22Description%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%7D%2C%7B%22id%22%3A%22Hashtag%22%2C%22displayName%22%3A%22Hashtag%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%7D%2C%7B%22id%22%3A%22Today%22%2C%22displayName%22%3A%22Today%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleSheets%22%2C%22typeVersion%22%3A4.5%2C%22position%22%3A%5B-800%2C-2220%5D%2C%22id%22%3A%22b6accbaa-c668-4213-9f0d-3b75404516f8%22%2C%22name%22%3A%22Append%20new%20row%22%2C%22credentials%22%3A%7B%22googleSheetsOAuth2Api%22%3A%7B%22id%22%3A%22BZe7pqnp1dJMcD3G%22%2C%22name%22%3A%22Google%20Sheets%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22documentId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5Byour_document_id%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%20posts%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%3Fusp%3Ddrivesdk%22%7D%2C%22sheetName%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gid%3D0%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%23gid%3D0%22%7D%2C%22filtersUI%22%3A%7B%22values%22%3A%5B%7B%22lookupColumn%22%3A%22Filename%22%2C%22lookupValue%22%3A%22%3D%7B%7B%20%24(%27Get%20filename%27).item.json.Filename%20%7D%7D%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleSheets%22%2C%22typeVersion%22%3A4.5%2C%22position%22%3A%5B-1460%2C-2020%5D%2C%22id%22%3A%22e2dc3d8e-6a89-43fa-8427-7dec68a11d02%22%2C%22name%22%3A%22Find%20deleted%20image%20row%22%2C%22credentials%22%3A%7B%22googleSheetsOAuth2Api%22%3A%7B%22id%22%3A%22BZe7pqnp1dJMcD3G%22%2C%22name%22%3A%22Google%20Sheets%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22strict%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%2243d2c6dd-05c6-4f6b-be66-0efa89f506be%22%2C%22leftValue%22%3A%22%7B%7B%20%24json.Status%20%7D%7D%22%2C%22rightValue%22%3A%22%22%2C%22operator%22%3A%7B%22type%22%3A%22string%22%2C%22operation%22%3A%22empty%22%2C%22singleValue%22%3Atrue%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.if%22%2C%22typeVersion%22%3A2.2%2C%22position%22%3A%5B-1240%2C-2020%5D%2C%22id%22%3A%2272b2e044-2e30-491b-b800-5573788b0b47%22%2C%22name%22%3A%22Check%20if%20already%20completed%22%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22delete%22%2C%22documentId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5Byour_document_id%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%20posts%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%3Fusp%3Ddrivesdk%22%7D%2C%22sheetName%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gid%3D0%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5Byour_document_id%5D%2Fedit%23gid%3D0%22%7D%2C%22startIndex%22%3A%22%3D%7B%7B%20%24json.row_number%20%7D%7D%22%7D%2C%22type%22%3A%22n8n-nodes-base.googleSheets%22%2C%22typeVersion%22%3A4.5%2C%22position%22%3A%5B-1020%2C-2020%5D%2C%22id%22%3A%224c3f01ff-afe7-4916-8004-4d345e330982%22%2C%22name%22%3A%22Delete%20row%22%2C%22credentials%22%3A%7B%22googleSheetsOAuth2Api%22%3A%7B%22id%22%3A%22BZe7pqnp1dJMcD3G%22%2C%22name%22%3A%22Google%20Sheets%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22assignments%22%3A%7B%22assignments%22%3A%5B%7B%22id%22%3A%22245d034e-843a-4a67-bd59-ea7142643c55%22%2C%22name%22%3A%22Filename%22%2C%22value%22%3A%22%3D%7B%7B%20%24json.path.replace(%27%2Fnas%2FInstagram%2F%27%2C%27%27)%20%7D%7D%22%2C%22type%22%3A%22string%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.set%22%2C%22typeVersion%22%3A3.4%2C%22position%22%3A%5B-1900%2C-2120%5D%2C%22id%22%3A%22be587136-3302-4926-9620-833740c43437%22%2C%22name%22%3A%22Get%20filename%22%7D%2C%7B%22parameters%22%3A%7B%22fileSelector%22%3A%22%3D%7B%7B%20%24(%27Local%20File%20Trigger%27).item.json.path%20%7D%7D%22%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.readWriteFile%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-1460%2C-2220%5D%2C%22id%22%3A%220ca43d84-6b0c-433a-a258-83964c184126%22%2C%22name%22%3A%22Load%20image%22%7D%2C%7B%22parameters%22%3A%7B%22assignments%22%3A%7B%22assignments%22%3A%5B%7B%22id%22%3A%22a50ce10d-3a05-41c2-b0d2-90dcf741aca0%22%2C%22name%22%3A%22content%22%2C%22value%22%3A%22%3D%7B%7B%20%24json.content.split(%5C%22%5C%5Cn%5C%5Cn%5C%22)%5B0%5D%20%7D%7D%22%2C%22type%22%3A%22string%22%7D%2C%7B%22id%22%3A%22245d034e-843a-4a67-bd59-ea7142643c55%22%2C%22name%22%3A%22hashtag%22%2C%22value%22%3A%22%3D%7B%7B%20%24json.content.split(%5C%22%5C%5Cn%5C%5Cn%5C%22)%5B1%5D%20%7D%7D%22%2C%22type%22%3A%22string%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.set%22%2C%22typeVersion%22%3A3.4%2C%22position%22%3A%5B-1020%2C-2220%5D%2C%22id%22%3A%225a99185f-3a69-4b8b-9b03-59a21f086fba%22%2C%22name%22%3A%22Split%20content%20and%20hashtag%22%7D%2C%7B%22parameters%22%3A%7B%22resource%22%3A%22image%22%2C%22operation%22%3A%22analyze%22%2C%22modelId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gpt-4o%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22GPT-4O%22%7D%2C%22text%22%3A%22Analyze%20the%20image%20and%20select%20a%20quote%2C%20aphorism%2C%20poem%20excerpt%2C%20or%20a%20statement%20by%20a%20famous%20person%2C%20in%20Italian%20or%20English%2C%20that%20best%20represents%20it.%20Preferably%2C%20the%20quote%20should%20relate%20specifically%20to%20the%20subject%20of%20the%20photo.%20Indicate%20the%20author%20and%20provide%20only%20the%20quote%2C%20without%20explanation.%20Additionally%2C%20identify%2015%20to%2030%20hashtags%20(preferably%20in%20English%20or%20Italian)%20for%20an%20Instagram%20post.%22%2C%22inputType%22%3A%22base64%22%2C%22options%22%3A%7B%22detail%22%3A%22auto%22%2C%22maxTokens%22%3A300%7D%7D%2C%22type%22%3A%22%40n8n%2Fn8n-nodes-langchain.openAi%22%2C%22typeVersion%22%3A1.7%2C%22position%22%3A%5B-1240%2C-2220%5D%2C%22id%22%3A%22f8b2c560-ad6f-4872-a18c-8ef568b8dec3%22%2C%22name%22%3A%22Analyze%20image%22%2C%22credentials%22%3A%7B%22openAiApi%22%3A%7B%22id%22%3A%22%5Byour_OpenAI_account%5D%22%2C%22name%22%3A%22OpenAi%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22content%22%3A%22%23%23%20Google%20Sheet%20management%20flow%5Cn**This%20flow%20automatically%20populate%20the%20google%20sheet%20leaving%20the%20date%20blank%20for%20preemptive%20user%20check%20and%20validation%22%2C%22height%22%3A460%2C%22width%22%3A1840%2C%22color%22%3A4%7D%2C%22type%22%3A%22n8n-nodes-base.stickyNote%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-2420%2C-2300%5D%2C%22id%22%3A%22da4e500e-42ae-47a5-b677-7524264bc5f7%22%2C%22name%22%3A%22Sticky%20Note%22%7D%5D%2C%22connections%22%3A%7B%22Local%20File%20Trigger%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Wait%201s%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Wait%201s%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Get%20filename%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Check%20if%20added%20or%20deleted%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Load%20image%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Find%20deleted%20image%20row%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Append%20new%20row%22%3A%7B%22main%22%3A%5B%5B%5D%5D%7D%2C%22Find%20deleted%20image%20row%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Check%20if%20already%20completed%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Check%20if%20already%20completed%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Delete%20row%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20filename%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Check%20if%20added%20or%20deleted%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Load%20image%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Analyze%20image%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Split%20content%20and%20hashtag%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Append%20new%20row%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Analyze%20image%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Split%20content%20and%20hashtag%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%7D%2C%22pinData%22%3A%7B%7D%7D' frame=true></n8n-demo>


### **Workflow Description**
1. **Trigger**:
   - A **Local File Trigger** monitors the folder `/nas/Instagram/` for changes (specifically, when files are added or deleted).
   - Since we are talking about a network folder, with some possible delays, check both `Await Write Finish` and `Use Polling`
   - Events like `add` or `unlink` (file addition or deletion) initiate the workflow.

2. **Debounce**:
   - A **Wait Node** delays the workflow for 1 second to ensure all file operations are stable before proceeding.

3. **Extract Filename**:
   - A **Set Node** extracts the filename from the file path, in order to reuse within the next nodes without appliyng always the same Javascript function.

4. **Determine Action**:
   - An **If Node** checks whether the file was added (`add`) or deleted (`unlink`).


#### **File Added**:
1. **Load Image**:
   - The workflow reads the binary added image file from disk.

2. **Analyze Image**:
   - The image is analyzed using OpenAI’s GPT model to generate:
     - A quote or caption relevant to the image.
     - 15–30 hashtags for Instagram posts.
    
    Here is the used prompt:
    ``` 
    Analyze the image and select a quote, aphorism, poem excerpt, 
    or a statement by a famous person, in Italian or English, 
    that best represents it. Preferably, the quote should relate specifically 
    to the subject of the photo. Indicate the author and provide only the quote, 
    without explanation. Additionally, identify 15 to 30 hashtags 
    (preferably in English or Italian) for an Instagram post.
    ``` 
   ! Image analysis is a rather resource-intensive activity in terms of tokens: processing a photo with the specified dimensions using this prompt and **GPT-4O** as the model costs approximately **1300-1500 tokens per call**, which translates to about **€0.01** per photo. To save costs, you can set `detail = low` to process a photo at just 512x512 pixels, though this comes with the risk of reduced detail and lower output quality. 

3. **Prepare Content**:
   - The analysis results are split into:
     - Description (quote/caption).
     - Hashtags.

4. **Append to Google Sheet**:
   - The image filename, description, and hashtags are appended to the previously defined Google Sheet. The `Date` column is not set: since It's the trigger for the publishing flows, the image and content will not be published until you will review it and define a publishing date.
  


#### **File Deleted**:
1. **Locate Row**:
   - The workflow searches the Google Sheet for the row corresponding to the deleted file’s filename.

2. **Check Completion**:
   - It verifies whether the file has already been handled or marked as completed.

3. **Delete Row**:
   - If that image has never been posted, the corresponding row in the Google Sheet is deleted.


<br/><br/>

# Step 5: Define the automatic posting process

This **n8n workflow** automates the process of creating and publishing daily Instagram posts. It uses image analyzer to handle both portrait and landscape images without cropping and at the end it ensures unnecessary files are deleted from both the local disk cloud ones.

Here's a detailed breakdown:


<n8n-demo workflow='%7B%22nodes%22%3A%5B%7B%22parameters%22%3A%7B%22rule%22%3A%7B%22interval%22%3A%5B%7B%22triggerAtHour%22%3A9%7D%5D%7D%7D%2C%22type%22%3A%22n8n-nodes-base.scheduleTrigger%22%2C%22typeVersion%22%3A1.2%2C%22position%22%3A%5B-2380%2C-1580%5D%2C%22id%22%3A%2253c184e3-d92b-427f-a5e2-0a830ae0eccf%22%2C%22name%22%3A%22Schedule%20Trigger%22%7D%2C%7B%22parameters%22%3A%7B%22documentId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5BDocument_ID%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%20posts%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5BDocument_ID%5D%2Fedit%3Fusp%3Ddrivesdk%22%7D%2C%22sheetName%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gid%3D0%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5BDocument_ID%5D%2Fedit%23gid%3D0%22%7D%2C%22filtersUI%22%3A%7B%22values%22%3A%5B%7B%22lookupColumn%22%3A%22Today%22%2C%22lookupValue%22%3A%22TRUE%22%7D%2C%7B%22lookupColumn%22%3A%22Status%22%7D%5D%7D%2C%22options%22%3A%7B%22outputFormatting%22%3A%7B%22values%22%3A%7B%22general%22%3A%22FORMATTED_VALUE%22%2C%22date%22%3A%22FORMATTED_STRING%22%7D%7D%2C%22returnFirstMatch%22%3Atrue%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleSheets%22%2C%22typeVersion%22%3A4.5%2C%22position%22%3A%5B-2120%2C-1580%5D%2C%22id%22%3A%22f179a159-5354-42e0-8f36-aab26e6a53df%22%2C%22name%22%3A%22Get%20today%20post%22%2C%22alwaysOutputData%22%3Afalse%2C%22credentials%22%3A%7B%22googleSheetsOAuth2Api%22%3A%7B%22id%22%3A%22BZe7pqnp1dJMcD3G%22%2C%22name%22%3A%22Google%20Sheets%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22jsCode%22%3A%22return%20items.map(file%20%3D%3E%20%7B%5Cn%20%20return%20%7B%20json%3A%20%7B%20%27link%27%3A%20%60https%3A%2F%2Fdrive.usercontent.google.com%2Fdownload%3Fid%3D%24%7Bfile.json.id%7D%26export%3Dview%26authuser%3D0%60%2C%20%27name%27%3A%20file.json.name%20%7D%20%7D%3B%5Cn%7D)%3B%22%7D%2C%22type%22%3A%22n8n-nodes-base.code%22%2C%22typeVersion%22%3A2%2C%22position%22%3A%5B-780%2C-1580%5D%2C%22id%22%3A%225d7d2f14-5df7-43f4-9fb1-118f76f35ca2%22%2C%22name%22%3A%22Create%20image%20link%22%7D%2C%7B%22parameters%22%3A%7B%22method%22%3A%22POST%22%2C%22url%22%3A%22https%3A%2F%2Fgraph.facebook.com%2Fv20.0%2F%5B%5BInstagram_Account_ID%5D%5D%2Fmedia%22%2C%22sendHeaders%22%3Atrue%2C%22headerParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22Authorization%22%2C%22value%22%3A%22%3DBearer%20%5BAccess_Token%5D%22%7D%5D%7D%2C%22sendBody%22%3Atrue%2C%22bodyParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22image_url%22%2C%22value%22%3A%22%3D%7B%7B%20%24json.link%20%7D%7D%22%7D%2C%7B%22name%22%3A%22caption%22%2C%22value%22%3A%22%3D%7B%7B%20%24(%27Get%20today%20post%27).item.json.Description%20%7D%7D%5Cn%5Cn%5Cn%5Cn%7B%7B%20%24(%27Get%20today%20post%27).item.json.Hashtag%20%7D%7D%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22id%22%3A%22073cfe16-0108-43cb-b030-dc7d7d264416%22%2C%22name%22%3A%22Create%20Instagram%20Media%20Container%22%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B-2120%2C-1240%5D%7D%2C%7B%22parameters%22%3A%7B%22method%22%3A%22POST%22%2C%22url%22%3A%22https%3A%2F%2Fgraph.facebook.com%2Fv20.0%2F%5B%5BInstagram_Account_ID%5D%5D%2Fmedia_publish%22%2C%22sendHeaders%22%3Atrue%2C%22headerParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22Authorization%22%2C%22value%22%3A%22%3DBearer%20%5BAccess_Token%5D%22%7D%5D%7D%2C%22sendBody%22%3Atrue%2C%22bodyParameters%22%3A%7B%22parameters%22%3A%5B%7B%22name%22%3A%22creation_id%22%2C%22value%22%3A%22%3D%7B%7B%20%24(%27Create%20Instagram%20Media%20Container%27).item.json.id%20%7D%7D%22%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22id%22%3A%223ed6dafb-7db5-415b-a1d0-c360229a1b2d%22%2C%22name%22%3A%22Publish%20Instagram%20Media%22%2C%22type%22%3A%22n8n-nodes-base.httpRequest%22%2C%22typeVersion%22%3A4.2%2C%22position%22%3A%5B-1680%2C-1240%5D%2C%22alwaysOutputData%22%3Afalse%2C%22retryOnFail%22%3Afalse%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22deleteFile%22%2C%22fileId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%3D%7B%7B%20%24(%27Upload%20image%20to%20Google%20Drive%27).item.json.id%20%7D%7D%22%2C%22mode%22%3A%22id%22%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleDrive%22%2C%22typeVersion%22%3A3%2C%22position%22%3A%5B-1460%2C-1240%5D%2C%22id%22%3A%2254a6b488-33b1-4c6b-8e2b-b6aeb3b3a4e1%22%2C%22name%22%3A%22Delete%20image%22%2C%22credentials%22%3A%7B%22googleDriveOAuth2Api%22%3A%7B%22id%22%3A%22Ym1lgVaQ3n7k1AQ2%22%2C%22name%22%3A%22Google%20Drive%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22amount%22%3A30%7D%2C%22id%22%3A%2200035520-58b0-4000-b8cb-d50714a0bf3c%22%2C%22name%22%3A%22Wait%2030s%22%2C%22type%22%3A%22n8n-nodes-base.wait%22%2C%22typeVersion%22%3A1.1%2C%22position%22%3A%5B-1900%2C-1240%5D%2C%22webhookId%22%3A%2258bd81e9-d956-42cb-8d7d-62ed066b9b2d%22%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22information%22%7D%2C%22type%22%3A%22n8n-nodes-base.editImage%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-1680%2C-1580%5D%2C%22id%22%3A%22b9f0e4d3-3ae1-4535-bf9c-357a447c2fa4%22%2C%22name%22%3A%22Get%20image%20info%22%7D%2C%7B%22parameters%22%3A%7B%22borderWidth%22%3A%22%3D%7B%7B%20(%24json.size.height%20-%20%24json.size.width)%20%2F2%20%7D%7D%22%2C%22borderHeight%22%3A0%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.editImage%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-1240%2C-1720%5D%2C%22id%22%3A%22356bbd1d-1ede-48c8-bc47-6ec956364599%22%2C%22name%22%3A%22Add%20border%20to%20portrait%20images%22%7D%2C%7B%22parameters%22%3A%7B%22name%22%3A%22%3D%7B%7B%20%24(%27Get%20today%20post%27).item.json.Filename%20%7D%7D%22%2C%22driveId%22%3A%7B%22__rl%22%3Atrue%2C%22mode%22%3A%22list%22%2C%22value%22%3A%22My%20Drive%22%7D%2C%22folderId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5BFolder_ID%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdrive.google.com%2Fdrive%2Ffolders%2F%5BFolder_ID%5D%22%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleDrive%22%2C%22typeVersion%22%3A3%2C%22position%22%3A%5B-1020%2C-1580%5D%2C%22id%22%3A%22c5929c43-e01b-42e7-8ea8-869a16a6bc8a%22%2C%22name%22%3A%22Upload%20image%20to%20Google%20Drive%22%2C%22credentials%22%3A%7B%22googleDriveOAuth2Api%22%3A%7B%22id%22%3A%22Ym1lgVaQ3n7k1AQ2%22%2C%22name%22%3A%22Google%20Drive%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22conditions%22%3A%7B%22options%22%3A%7B%22caseSensitive%22%3Atrue%2C%22leftValue%22%3A%22%22%2C%22typeValidation%22%3A%22strict%22%2C%22version%22%3A2%7D%2C%22conditions%22%3A%5B%7B%22id%22%3A%22d0fab124-b492-4d40-aeeb-4dd2c139b1f7%22%2C%22leftValue%22%3A%22%3D%7B%7B%20%24json.size.height%20%7D%7D%22%2C%22rightValue%22%3A%22%3D%7B%7B%20%24json.size.width%20%7D%7D%22%2C%22operator%22%3A%7B%22type%22%3A%22number%22%2C%22operation%22%3A%22gt%22%7D%7D%5D%2C%22combinator%22%3A%22and%22%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.if%22%2C%22typeVersion%22%3A2.2%2C%22position%22%3A%5B-1460%2C-1580%5D%2C%22id%22%3A%22d10bc224-89d2-4bb6-abba-650c58aacb99%22%2C%22name%22%3A%22Check%20if%20portrait%20or%20landscape%22%7D%2C%7B%22parameters%22%3A%7B%22fileSelector%22%3A%22%3D%2Fnas%2FInstagram%2F%7B%7B%20%24json.Filename%20%7D%7D%22%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.readWriteFile%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-1900%2C-1580%5D%2C%22id%22%3A%2217fd2144-b228-4896-a7b5-80d2825bb0b1%22%2C%22name%22%3A%22Open%20image%20from%20disk%22%7D%2C%7B%22parameters%22%3A%7B%22operation%22%3A%22update%22%2C%22documentId%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22%5BDocument_ID%5D%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%20posts%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5BDocument_ID%5D%2Fedit%3Fusp%3Ddrivesdk%22%7D%2C%22sheetName%22%3A%7B%22__rl%22%3Atrue%2C%22value%22%3A%22gid%3D0%22%2C%22mode%22%3A%22list%22%2C%22cachedResultName%22%3A%22Instagram%22%2C%22cachedResultUrl%22%3A%22https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F%5BDocument_ID%5D%2Fedit%23gid%3D0%22%7D%2C%22columns%22%3A%7B%22mappingMode%22%3A%22defineBelow%22%2C%22value%22%3A%7B%22Filename%22%3A%22%3D%7B%7B%20%24(%27Get%20today%20post%27).item.json.Filename%20%7D%7D%22%2C%22Status%22%3A%22%3DDone%20%7B%7B%20%24now.setZone(%27Europe%2FRome%27).toFormat(%27yyyy-MM-dd%20HH%3Amm%3Ass%27)%20%7D%7D%22%7D%2C%22matchingColumns%22%3A%5B%22Filename%22%5D%2C%22schema%22%3A%5B%7B%22id%22%3A%22Date%22%2C%22displayName%22%3A%22Date%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%2C%7B%22id%22%3A%22Filename%22%2C%22displayName%22%3A%22Filename%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Afalse%7D%2C%7B%22id%22%3A%22Description%22%2C%22displayName%22%3A%22Description%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%2C%7B%22id%22%3A%22Hashtag%22%2C%22displayName%22%3A%22Hashtag%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%2C%7B%22id%22%3A%22Today%22%2C%22displayName%22%3A%22Today%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Atrue%7D%2C%7B%22id%22%3A%22Status%22%2C%22displayName%22%3A%22Status%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22removed%22%3Afalse%7D%2C%7B%22id%22%3A%22row_number%22%2C%22displayName%22%3A%22row_number%22%2C%22required%22%3Afalse%2C%22defaultMatch%22%3Afalse%2C%22display%22%3Atrue%2C%22type%22%3A%22string%22%2C%22canBeUsedToMatch%22%3Atrue%2C%22readOnly%22%3Atrue%2C%22removed%22%3Atrue%7D%5D%7D%2C%22options%22%3A%7B%7D%7D%2C%22type%22%3A%22n8n-nodes-base.googleSheets%22%2C%22typeVersion%22%3A4.5%2C%22position%22%3A%5B-1240%2C-1240%5D%2C%22id%22%3A%22ccbf6cb2-fbf8-4b9d-ab17-72616f517aa8%22%2C%22name%22%3A%22Mark%20row%20as%20Done%22%2C%22credentials%22%3A%7B%22googleSheetsOAuth2Api%22%3A%7B%22id%22%3A%22BZe7pqnp1dJMcD3G%22%2C%22name%22%3A%22Google%20Sheets%20account%22%7D%7D%7D%2C%7B%22parameters%22%3A%7B%22command%22%3A%22%3Drm%20%2Fnas%2Finstagram%2F%7B%7B%20%24(%27Get%20today%20post%27).item.json.Filename%20%7D%7D%22%7D%2C%22type%22%3A%22n8n-nodes-base.executeCommand%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-1020%2C-1240%5D%2C%22id%22%3A%22d245c276-2631-4896-9861-148b68f8e08b%22%2C%22name%22%3A%22Remove%20image%20from%20disk%22%7D%2C%7B%22parameters%22%3A%7B%22content%22%3A%22%23%23%20Instagram%20post%20generator%5Cn**This%20scheduled%20flow%20create%20the%20instagram%20post%20for%20the%20daily%20image%20specified%20in%20the%20excel%20sheet%22%2C%22height%22%3A760%2C%22width%22%3A1840%2C%22color%22%3A6%7D%2C%22type%22%3A%22n8n-nodes-base.stickyNote%22%2C%22typeVersion%22%3A1%2C%22position%22%3A%5B-2420%2C-1800%5D%2C%22id%22%3A%22134b776c-3a54-4437-933c-95c7ca421283%22%2C%22name%22%3A%22Sticky%20Note1%22%7D%5D%2C%22connections%22%3A%7B%22Schedule%20Trigger%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Get%20today%20post%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20today%20post%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Open%20image%20from%20disk%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Create%20image%20link%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Create%20Instagram%20Media%20Container%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Create%20Instagram%20Media%20Container%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Wait%2030s%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Publish%20Instagram%20Media%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Delete%20image%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Delete%20image%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Mark%20row%20as%20Done%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Wait%2030s%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Publish%20Instagram%20Media%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Get%20image%20info%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Check%20if%20portrait%20or%20landscape%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Add%20border%20to%20portrait%20images%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Upload%20image%20to%20Google%20Drive%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Upload%20image%20to%20Google%20Drive%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Create%20image%20link%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Check%20if%20portrait%20or%20landscape%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Add%20border%20to%20portrait%20images%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%2C%5B%7B%22node%22%3A%22Upload%20image%20to%20Google%20Drive%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Open%20image%20from%20disk%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Get%20image%20info%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%2C%22Mark%20row%20as%20Done%22%3A%7B%22main%22%3A%5B%5B%7B%22node%22%3A%22Remove%20image%20from%20disk%22%2C%22type%22%3A%22main%22%2C%22index%22%3A0%7D%5D%5D%7D%7D%2C%22pinData%22%3A%7B%7D%7D' frame=true></n8n-demo>


### **Workflow Description**

1. **Schedule Trigger**: 
   - Activates the workflow daily at 9 AM.

2. **Get Today Post (Google Sheets)**
   - Fetches the row marked as "Today" (`Today` column = `TRUE`) from the previously defined Google sheet.
   - It returns only first row since making two simultaneous posts don't give you any benefit.

3. **Open Image from Disk**
   - Reads the image file associated with today's post from the NAS shared folder mounted in n8n server.

4. **Get Image Info**, **Check if Portrait or Landscape**
   - Analyzes the image dimensions to check its orientation (portrait or landscape).
   - If portrait, proceeds to add borders to make it square.

5. **Add Border to Portrait Images**
   - Adds borders to portrait images to make them square for Instagram and avoid cropping of 3:2 or 4:3 format pictures.

6. **Upload Image to Google Drive**, **Create Image Link**
   - Facebook Graph API doesn't allow binary image data, but only urls, so we need to:
     - Upload the processed image to Google Drive for hosting.
     - Generate a publicly accessible link for the image hosted on Google Drive, by using the Unique ID of the just published file.

7.  **Create Instagram Media Container (HTTP Request)**
   - Sends the image link and caption to Instagram's API to create a media container.
   - Replace `[Instagram_Account_ID]` with the ID of your IG account you copied from Step 3.3
   - Replace  `[Access_Token]` with the token generated at the Step 3.5 (Remember to keep the string `Berarer ` before the token)
   - Combine the description and hashtags from the Google Sheet as the caption for the picture.

8.  **Wait 30s**
    - Ensures the media container is ready before publishing.

9.  **Publish Instagram Media (HTTP Request)**
    - ublishes the media container created in the previous step.
    - Requires the `creation_id` from the media container.

10. **Delete Image**, **Remove Image from Disk**
    - Deletes the image from Google Drive after publishing.
    - Removes the image file from the local disk to clean up.

11. **Mark Row as Done (Google Sheets)**
    - Updates the Google Sheet to mark the post as completed, by flagging the `Status` column with `Done` plus the timestamp of the operation for providing a complete information.



<br/><br/>

# Step 6: Let's try it!
Now let's try the entire process:

## 1. Select the photos
I usually edit photos in Lightroom, and the ones I want to publish are exported to the network share configured earlier and accessible by n8n. Following my usual workflow for Instagram, I export the photos in JPEG format, maximum quality, with a short side resolution of 1080px, 240 dpi, and sharpening set for screen.

## 2. The post definition process prepare the content
This is handled "under the hood" by the first defined workflow and triggered by the export on the defined folder, so you actually don't need to do anything.

## 3. Review post content
- Access to the Google sheet and you will find it automatically populated by AI
- Review the suggested description and hashtags and modify them as you wish 
- Set the desired publishing date for each image (I assumed that the flow will process only a single image a day)

## 4. Wait for it!
At the scheduled time of the set day, the second process will automatically publish the post on Instagram, remove the useless picture both from local drive and Google Drive, and update the spreadsheet.



<br/><br/>

# Step 7: Enjoy
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