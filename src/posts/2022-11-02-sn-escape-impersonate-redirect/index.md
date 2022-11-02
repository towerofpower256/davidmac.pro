---
title: "How to un-impersonate in ServiceNow when locked in the service portal"
description: Don't you hate it when you impersonate a user in ServiceNow, but you can't un-impersonate because a redirect keeps you stuck in the service portal? Here's a quick way to escape the redirect prison.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2022-11-02
tags:
- ServiceNow
eleventyExcludeFromCollections: false
---

So you've impersonated a user in ServiceNow, but there's a redirect that forces that user into the service portal and you can't end the impersonation. How do you un-impersonate and get back to your own admin user?

In the platform UI, you can just click on the user icon in the top-right then click on "End impersonation".

[![Platform UI, end impersonation](platform-end-impersonation.png)](platform-end-impersonation.png)

Sadly, this option is not available in the service portal, which doesn't help if you're stuck in there.

[![Service Portal, no end impersonation](sp-no-end-impersonate.png)](sp-no-end-impersonate.png)

## Option 1 - log out
The easy option is to log out and back in again, but this can get annoying and time-consuming, especially when there's multi-factor authentication to do and big passwords to remember. 

Do that 30 times in a row, and it's enough to make you dread system testing.

Surely there's a better way... right?

## Option 2 - impersonate_dialog.do
Simply change the URL to **"/impersonate_dialog.do"**. That'll take you back to the impersonation dialogue, and you can go end the impersonation by double-clicking on your own user without having to log out and back in again. 

It's quick and easy, and a lot faster than logging out and back in. Magic!

[![impersonate_dialog.do](impersonate_dialog_do.png)](impersonate_dialog_do.png)

## Option 3 - SN Utils /unimp
The popular browser plugin **SN Utils** has a shortcut to stop impersonating a user. It's available for Chrome, FireFox, Edge, and Safari. 

You can get it here: www.arnoudkooi.com

Even from the service portal, just use the slash command **/unimp** to end your impersonation.

[![SN Utils /unimp](sn-utils-unimp.png)](sn-utils-unimp.png)


