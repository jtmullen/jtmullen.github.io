---
layout: post
title: Syncing a Google Sheets Table with a Wordpress Table
subtitle: American Solar Challenge Team Status Board
author: Jonathan Mullen
categories: ief
tags:
  - solar car
  - wordpress
  - google apps script
---

## Why do this?

I volunteer on the staff of the [American Solar Challenge](https://www.americansolarchallenge.org/) where one of the things I do is maintain the website. Prior to the event, the teams are required to submit various paperwork, fees, and reports in order to participate in the event. The various staff responsible for reviewing these submissions keep track of these submissions in a shared google drive with a google sheet tracking the overall status. The status information is published publicly for teams to see their status and which competitors are registering to compete. The website is hosted on wordpress. So the goal is to keep this up to date regularly without me in the loop. Otherwise the other staff get emails asking why the website is not showing the corret status. 

## How to accomplish this?

To accomplish this some code will need to be running on a server to do the update. Since the data is coming from a Google Sheets spreadsheet an obvious choice is [Google Apps Script](https://developers.google.com/apps-script). This is a scripting tool for google drive that allows you to interact with various documents. Our setup will be one of the more simple setups where a single script needs to interact with a single document, so the script can be created from the Extensions -> Apps Script menu in this document. The google apps script will read the data from the spreadsheet and turn it into what we want to display on wordpress. 

## Getting it into Wordpress

The [Wordpress REST API](https://developer.wordpress.org/rest-api/) will be used to update the data on the wordpress page. The api needs to be enabled from wordpress and potentially with your hosting provider to be used. Once you have done this you should navigate to Users -> Profile in the wordpress dashboard and scroll down to create an application password. This is the password to use to access the API in conjunction with your normal username. 

In Wordpress we will also need to create the page, if it doesn't already exist, and note down the page ID. To get this you can open the page editor and look in the url which will have the query string `post=####` where #### is the page ID. 

We will also need to prepare the page for the update. In my case, we only want to update a segment of the page. In order to track where this segment is I have used two HTML comments to mark the start and end of the page sement which we should update the content within:

```html
<!-- AUTO GEN CONTENT -->
 [CONTENT TO UPDATE]
 <!-- AUTO GEN CONTENT -->
```
_Note: The Wordpress Block Editor has a habit of breaking pages updated this way. I always use the classic editor or code editor to edit these pages._

Then in google apps script we will use these comments to define where to put our generated content. If you have multiple segments in the page to update, simply use different comments for each. I also usually create the content I want to generate first in wordpress so I can just copy and paste all the styling rather than writing it out myself. At this point we have what we need from wordpress and can get to work in the google apps script. 

## Implementation

Back in our Google Apps Script Project we will have a file called code.gs. Feel free to rename it, the name is not important. At the top we will start with some constants needed:

```js
// Wordpress Page / Login Info
const page_id = id
const api_url = 'https://americansolarchallenge.org/wp-json/wp/v2/pages/[id]'
const wordpress_user = '[username]'
const table_identifier = "<!-- AUTO GEN CONTENT -->"
```


### Content Template

To more easily create our HTML content we will use Google Apps Script [Templated HTML](https://developers.google.com/apps-script/guides/html/templates) feature. In the Google Apps Script editor create a new HTML file, I named it `table.html`. Templated HTML allows you to combine html with google apps script using the special html tag `<? ?>`. Content within the tag is evaluated as apps script while the rest is just html that will be directly put into the output. To create the below file I started by creating the table how I wanted it to look in wordpress between my comment tags. I then copied the html (via the Wordpress html editor) into this html file and added my google apps script on top. 

```html
<?
const hex_green = '#32cd32';
const hex_red = '#ff0000';
const hex_yellow = '#ffff00';
const hex_blue = '#0000ff';
const hex_grey = '#808080';
const hex_white = '#ffffff';
?>
<h3>Status Board</h3>
<figure class="wp-block-table">
<table style="width:100%">
<tr>
  <th>TEAM</th><th>CLASS</th>
  <? for ( var i = 0; i < numitems; i++){ ?>
  <th> <?=i?> </th>
  <? } ?>
</tr>
<? for (let i = 0; i < teams.length - 1; i++) {
  team_data = teams[i+1];
  if(!team_data[1]){
    // skip empty row
    continue;
  }
  ?>
  <tr>
    <td style="white-space:nowrap;"><a href="<?=team_data[2]?>"><?=team_data[1]?></a></td>
    <td style="text-align: center;"><?=team_data[0]?></td>
    <? for( let j = 3; j < (num_items + 3); j++){
      status = team_data[j];
      color = hex_white;
      extracss = ""
      celltext = ""
      if(status.includes("GN")){
        color = hex_green;
        celltext = "G";
      }else if(status.includes("YE")){
        color = hex_yellow;
        celltext = "Y";
      }else if(status.includes("BU")){
        color = hex_blue;
        celltext = "B";
        extracss = "color: white;border-color:#4b4f58;"
      }else if(status.includes("RD")){
        color = hex_red;
        celltext = "R";
        extracss = "color: white;border-color:#4b4f58;"
      }else if(status.includes("GY")){
        color = hex_grey;
      }

      ?>
      <td bgcolor="<?=color?>" title="<?=teams[0][j]?>" style="text-align: center;<?!=extracss?>"><?=celltext?></td>

    <?}?>
    </tr>
<?}?>
</table></figure>
```

Lets go through this piece by piece. First we have some const definitions, these are in the special google apps script tag since they are variables we want to use in our google apps script parts below. 
```html
<?
const hex_green = '#32cd32';
const hex_red = '#ff0000';
const hex_yellow = '#ffff00';
const hex_blue = '#0000ff';
const hex_grey = '#808080';
const hex_white = '#ffffff';
?>
```

Next we have our header and the start of our table. These are just html since we want them recreated exactly in the final product: 
```html
<h3>Status Board</h3>
<figure class="wp-block-table">
<table style="width:100%">
  ```

Within the table, the first thing we create is our header row. In my case this is two static columns followed by the numbers 1-X where X is definied from the spreadsheet. The `numitems` variable is definied in our Google Apps Script and passed when the template is evaluated, this will be shown later. 
```html
<tr>
  <th>TEAM</th><th>CLASS</th>
  <? for ( var i = 0; i < numitems; i++){ ?>
  <th> <?=i?> </th>
  <? } ?>
</tr>
```

Next we will iterate through every team and create the row with their status. `teams` is another variable we will pass to the template. It is an array of all the teams and their status information. Within our for loop we first extract the information for the team we are processing. We then skip the iteration if the row is empty - sometimes rows end up empty when a team drops out or extra empty rows exist at the end of the table, so we just want to skip that so the empty row doesn't make it to the website. 

```html
<? for (let i = 0; i < teams.length - 1; i++) {
  team_data = teams[i+1];
  if(!team_data[1]){
    // skip empty row
    continue;
  }
  ?>
```

Now that we have a non-empty team row we will create the row for the team. We have defined our first two columns to show the team name (which will link to their website) and the class they are competing in. This data can be found in our teams array in the first 3 columns, so first we put this data in place. Next we loop through the third to final columns in order to place the statuses. This is very specific to our setup, I am simply translating the text used in dropdowns on the spreadsheet ("GN", "YE", "BU", etc) to the text we want in the table cell on the website as well as the color of the cells. Once we determine this info we create the actual `<td>` table data cell and continue on to the next. 
```html
  <tr>
    <td style="white-space:nowrap;"><a href="<?=team_data[2]?>"><?=team_data[1]?></a></td>
    <td style="text-align: center;"><?=team_data[0]?></td>
    <? for( let j = 3; j < (num_items + 3); j++){
      status = team_data[j];
      color = hex_white;
      extracss = ""
      celltext = ""
      if(status.includes("GN")){
        color = hex_green;
        celltext = "G";
      }else if(status.includes("YE")){
        color = hex_yellow;
        celltext = "Y";
      }else if(status.includes("BU")){
        color = hex_blue;
        celltext = "B";
        extracss = "color: white;border-color:#4b4f58;"
      }else if(status.includes("RD")){
        color = hex_red;
        celltext = "R";
        extracss = "color: white;border-color:#4b4f58;"
      }else if(status.includes("GY")){
        color = hex_grey;
      }
      ?>
      <td bgcolor="<?=color?>" title="<?=teams[0][j]?>" style="text-align: center;<?!=extracss?>"><?=celltext?></td>

    <?}?>
    </tr>
<?}?>
```

Finally, we close the table html tags. 
```html
</table></figure>
```

### Getting the Executed Template

Once our template is done we can use it in our Google Apps Script code as shown below. We have determined the number of items (our array width) and extracted the team data from the spreadsheet. So we create an instance of the template, pass our data, and then evaluate the template with that data. This function will return the html for a completed table with our passed data. 
```js
function generate_table(sheet, num_items){

  var template = HtmlService.createTemplateFromFile('table.html');
  template.numitems = num_items;
  template.teams = sheet;
  return template.evaluate().getContent();
 
}
```

### Connecting to Wordpress

There are two things we need to be able to do to update the content in our page. First, we need to get the latest page content (so we can change it) and then we need to push our updates to the site. We will do this with Google Apps Scripts [UrlFetchApp](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app) which can be used to make simple HTTP requests. 

First, we need the HTTP header for our requests. This is simply the authorization plus some info about our request. _Note: Google Apps Script doesn't have a great way to manage secrets like passwords/api keys. You can use [Properties Service](https://developers.google.com/apps-script/reference/properties) which is a little better than putting it right in the code._
```js
  token = Utilities.base64EncodeWebSafe(wordpress_user+":"+wp_password);
  header = {
    'Authorization': 'Basic '+token, 
	  "User-Agent" : "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36",
	  "Accept" : "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
	  "Accept-Encoding" : "gzip, deflate, br",
	  "Accept-Language" : "en-US,en;q=0.5"
    };
```

To get the page content we just do a simple get request (remember, our url specifies which page we are interacting with). 
```js
function getPageContent(){
  // Get Current Page
  var get_options = {
    'method' : 'get',
    'headers' : header
  };
  var response = UrlFetchApp.fetch(api_url, get_options);
  var page_content = JSON.parse(response.getContentText()).content.rendered;
  return [response.getResponseCode(), page_content];
}
```

Updating the page content is a little more complex. This is a post request which requires the additional data with the request. Specifically, we need to tell wordpress that we want to edit the page and provide the new content. 
```js
function updatePageContent(content){
  var data = {
    'context': 'edit',
    'content' : content
  };

  var post_options = {
    'method' : 'post',
    'headers' : header,
    'payload' : data
  };

  var response = UrlFetchApp.fetch(api_url, post_options);
  return response.getResponseCode();
}
```

### Putting it together

Alright, so now we have all the pieces individually. We just need to put it together. Our function might look something like this: 

```js
function updateTable(){

  // First, get our spreadsheet data
  var spreadsheet = SpreadsheetApp.getActive(); // Get the spreadsheet
  var sheet_data = spreadsheet.getSheetByName("[sheetname]").getDataRangle().getValues(); // Get the data from the correct sheet
  var num_items = [number]; // Indicate how many items there are. In my case this came from the spreadsheet, up to you depending on use case
  
  // Next, Evaluate the Template
  new_table = generate_table(sheet_data, num_items);
  
  // Get the current page content
  var current_content = getPageContent();
  
  // Create the new content
  new_table = table_identifier + new_table + table_identifier; // Make sure new content has the identifiers
  var regexpr = table_identifier + "?(.*?)" + table_identifier; // Create Regular Expression Match String
  var new_content = current_content.replace(regexpr, new_table);  // Create new content using regex replacement
  
  // Update the content on the site
  updatePageContent(new_content);
}

```

### Triggers

Lastly, we need to define when this will run. In my use case we want two options - it will automatically run at night a few times a week (I adjust this as the event gets closer eventually automatically updating at least once per day) and be manually triggerable in the spreadsheet. For the manual trigger we will add a menu button by adding the following function: 

```js
// Runs when the spreadsheet starts, adds a tab at the top
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('Publish to Website 📢')
    .addItem('Update Table', 'updateTable')
    .addToUi();
}
```
Google Apps Script will automatically run the `onOpen()` function when the spreadsheet is opened and a new tab will be added to the menu with one dropdown item users can click called "Update Table". When clicked this will run our function `updateTable()`. _Note: The first time a user runs this they will need to click accept/OK on some prompts allowing the script to access the data via their account. They will then have to click run again before it actually runs_

For our second trigger we navigate to the Triggers page in Google Apps Script and add a new time driven trigger. Select the interval you want and link it to the `updateTable()` function. 

## Wrap Up

This was an overview of a generalized version of what I have implemeted to update the table on the [ASC/FSGP Pre-Event Status Board](https://www.americansolarchallenge.org/the-competition/2025-formula-sun-grand-prix/fsgp-2025-pre-event-team-status/). Wordpress and Google Sheets were already in use and their use is ingrained in the workflows of the organization so changing either is a lot of work - a lot more work than connecting them together like this. It is not the cleanest solution, has some limitations (don't edit the page in the block editor!), but it works most of the time, saves time, and gets the info out faster. 