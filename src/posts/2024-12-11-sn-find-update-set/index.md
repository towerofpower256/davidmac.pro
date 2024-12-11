---
title: "How to Find the Update Set That Changed Something in ServiceNow"
description: Have you wanted to find the update set something was changed in, but couldn't find it? A colleague has run into a problem where they've been asked to promote changes that someone has done for a catalog item, but they don't know the update set the changes were made in. Another colleague has run into a business rule that was changed, but they don't know the update set that it was changed in. Here's a quick guide on how to find the update set that changed something.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-12-11
tags:
- ServiceNow
eleventyExcludeFromCollections: false
---

## Update set entries (sys_update_xml) 
An **Update set** is made of **Customer updates** which is the snapshots of record as they get changed. 

[![Update set form with updates](update-set-with-updates.png)](update-set-with-updates.png)

To search for changes to a record: 
1. Copy the sys_id of the record you want to find update sets for (except for changes to a field [sys_dictionary], see below). 
1. Open a list of "Customer updates" [sys_update_xml] by typing "sys_update_xml.list" into the navigator search then pressing ENTER. 
1. Apply a filter to search for changes. 

If it's **not** for a field on a table, search "**Name ENDS WITH (sys_id of your record)**".

[![Update search by sys_id](update-search-record.png)](update-search-record.png)

If It's for a **field** on a table [sys_dictionary], search "**Name ENDS WITH (field name)**".

[![Update search by field name](update-search-field.png)](update-search-field.png)

If in doubt, try searching the "Target name" field. You just might find the answer you're looking for. 

[![Update search by name](update-search-name.png)](update-search-name.png)

## Version (sys_update_version) 
A Version [sys_update_version] is created whenever a configuration is changed. It helps capture a version of the record including a snapshot of the record, and you can even revert back to a specific version. 

We can use them to try to find the update set related to the change was made. 

Here's a screenshot of the **Versions** related list at the bottom of the default Business Rule form. Note the **Source** column which is the update set the version was captured in. 

[![List of versions for a business rule](version-list.png)](version-list.png)