# Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate
## Introduction
In this article we will consume WHO RSS feed and generate SharePoint News Pages. For that we will use power automate flow which will run every day and generate SharePoint news page for us.
## Output
![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/output.png?raw=true)

## Steps: 
- Link of WHO RSS Feed: https://www.who.int/rss-feeds/news-english.xml 
- I am assuming that you should have basic knowledge of SharePoint and power automate. 

### Step1: 
- Create new SharePoint communication site. 

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/1.png?raw=true)

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/2.png?raw=true)

### Step2: 

- Create new News Post in SharePoint communication site for template purpose so we can take post structure metadata of it.  

- To take metadata one way is to open browser dev tool(inspect element) before publishing template post and then publish post so you will find one request in Network tab named as “Save Page” which contains the request payload in json format.  

- Copy it in some notepad file to use it in letter on. Please see below screenshots so you can understand it easily. 

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/3.png?raw=true)

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/4.png?raw=true)

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/5.png?raw=true)

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/6.png?raw=true)

- At finally I have renamed post as WHOTemplate.aspx 

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/7.png?raw=true)

### Step3: 

- Now it’s time to create Power Automate flow which runs daily and checks for new RSS feeds and create SharePoint news post accordingly. 

![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/8.png?raw=true)

### Step4: 

- Below are the steps which consume WHO RSS feed and check for the posts created today and generate news post using our created WHOTemplate.aspx. 

#### 1 
Recurrence -> daily 
#### 2
Initialize variable Today -> formatDateTime(utcNow(),'yyyy-MM-dd') 
#### 3 
List all RSS feed items -> https://www.who.int/rss-feeds/news-english.xml 
#### 4 
Apply to each -> Repeat for all items in RSS Feed 
#### 5 
Condition if RSS feed item published today -> indexOf(items('Apply_to_each')?['publishDate'],variables('Today')) 
#### 6 
Step 5 – if YES
#### 7 
Compose JSON compatible file content (replacing " with \\\" because it breaks json) -> replace(uriComponent(items('Apply_to_each')?['summary']), '%22', '%5C%5C%5C%22') 
#### 8 
Compose removed new lines (removing new lines) -> uriComponentToString(replace(replace(outputs('Compose_JSON_compatible_file_content'), '%0D%0A', ''), '%0A', '')) 
#### 9 
Send an HTTP request to create file -> 
```_api/web/getfilebyserverrelativeurl('/sites/WHONews/SitePages/WHOTemplate.aspx')/copyto(strnewurl='/sites/WHONews/SitePages/@{items('Apply_to_each')?['title']}.aspx',bOverwrite=true) 
```
#### 10 
Get file metadata of newly created file ->
```
SitePages%2f @{items('Apply_to_each')?['title']}.aspx 
```
#### 11 
Send an HTTP request to checkout newly created file ->
```
_api/SitePages/pages(@{outputs('Get_file_metadata_of_newly_created_file')?['body/ItemId']})/checkoutpage 
```
```
{ 
  "content-type": "application/json;odata=verbose", 
  "Accept": "application/json;odata=verbose" 
} 
```
#### 12 
Send an HTTP request to SharePoint to add RSS feed content to file ->
```
_api/SitePages/pages(@{body('Get_file_metadata_of_newly_created_file')?['ItemId']})/SavePageAsDraft 
```
```
{ 
  "content-type": "application/json;odata=verbose", 
  "Accept": "application/json;odata=verbose" 
} 
```
-> Put meta data and replace the content where you need dynamic content please see screenshot 
#### 13 
Send an HTTP request to SharePoint for publish newly created file ->
```
_api/SitePages/pages(@{body('Get_file_metadata_of_newly_created_file')?['ItemId']})/Publish 
```
```
{ 
  "content-type": "application/json;odata=verbose", 
  "Accept": "application/json;odata=verbose" 
} 
```
![alt text](https://github.com/myvsparth/Generate-SharePoint-News-Posts-using-WHO-RSS-Feed-using-Power-Automate/blob/master/9.png?raw=true)

- Save and Run the Flow. 
