---
title: 
description: 
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-03-04
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
series: sn-theming
---

## Instance favicon
You can set a unique favicon / page icon for each instance which helps users identify the instance just be glancing at the browser tab.

> **Note:** Custom favicons are not visible on the Service Portal.

Here's how it looks out of the box.

[![Out of the box favicon](screenshot-favicon-ootb.png)](screenshot-favicon-ootb.png)

And here's how it looks with custom icons.

[![Custom favicon](screenshot-favicon-custom.png)](screenshot-favicon-custom.png)

First, add the image to ServiceNow.
1. **In production** navigate to "**System UI &gt; Images > Images**" or the table Images [db_image].
1. Click on "**New**".
1. Enter a "**Name**" for your image. Valid names must end in .gif, .png, .jpg, .ico, or .bmp. I'd recommend "favicon_(instance name)" e.g. "favicon_dev.png".
1. Click the "**Click to add**" link in the Image field, then select and upload the image.
1. Click "**Update**" to save the new image.
1. Repeat the above steps for as many unique favicon images you need (e.g. 1 for DEV, 1 for TEST).
1. Get the new Image [db_image] to the other instances either by doing a clone, or repeating the above steps in the other instances.

> **Why did we add the images in production?**
> While the system property that specifies which image to use will survive the clone, the image itself will not. The Image [db_image] table has no clone data preserves. This means images for all of the instances must be in production so that they come down in a clone.
>
> This also allows you to update the non-production favicon images from a single spot, so this isn't a bad thing.

Then, set up ServiceNow to use the new favicon image.
1. Open the instance you want to change the favicon for.
1. Navigate to "**System Properties &gt; System**".
1. Change the field "**Icon image displayed in the bookmarks and browser address bar**" [glide.product.icon], enter the file name of the uploaded image.
1. Click "**Save**".

Fun fact, "**glide.product.icon**" is one of the system properties preserved by the clone because it is considered part of the "theme". 

## Different icon for each instance
It's common to want a different favicon for each of your instances, from PROD, through TEST, to DEV.

> Rule of thumb
> Add the **content** to production and add **configuration** to non-production.

Add all of your favicons to the image database in your **production** instance so they will be included in clones, and then configure the system properties [glide.product.icon] in your non-production instances to each use a different icon.