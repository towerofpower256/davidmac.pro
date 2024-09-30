---
title: Non-Production Instance Theming Part 4 - Banner Colour UI16
description: Are you developing in production without realising? Did you accidentally delete that production data, or load test data into your live instance? It's easy to forget which ServiceNow instance you are in. Giving all of your ServiceNow instances an eye-catching coloured banner will be a constant reminder of which instance you are in.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-03-04
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
series: sn-theming
---

### UI16 Banner Theming
Theming the banner in UI16 is easy, and can be done using system properties.

When it comes to making the instances look unique, I recommend changing the colour of the banner border, the thin line that sits on the bottom of the banner. If this line is visually striking, the difference between instances can be obvious.

> **Note:** The banner colouring will **not** be visible if the user has selected their own theme.

1. Navigate to "**System Properties &gt; Basic Configuration UI16**".
1. Find "**Header divider stripe color**" [css.$navpage-header-divider-color] and set it to a new colour (e.g. #FF0000 for red).
1. Click on "**Save**".

This approach works the same for any of the other system properties that customise the UI16 theme, however a banner with an eye-catching colour is typically enough on its own.

Here are some variations of instance-unique banners.

[![UI16 banner theming](ui16-banner-theming-prod.png)](ui16-banner-theming-prod.png)

[![UI16 banner theming](ui16-banner-theming-test.png)](ui16-banner-theming-test.png)

[![UI16 banner theming](ui16-banner-theming-dev.png)](ui16-banner-theming-dev.png)