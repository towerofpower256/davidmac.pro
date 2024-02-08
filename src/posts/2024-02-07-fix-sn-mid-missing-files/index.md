---
title: How to recover a MID server after a failed upgrade
description: What happens a ServiceNow MID server tries to auto update itself but gets interrupted mid-upgrade and fails? You get a broken MID server that's missing half of its files, that's what. Here's how to recover a MID sever when the auto upgrade fails and it deletes its own files.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-02-07
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
---

## The problem
I ran into an interesting issue where a ServiceNow MID server was missing almost all of its files. Especially those in the **./agent/bin/** folder, they were all gone.

It turns out the MID server trued to update itself but a file security system had prevented parts of the MID server self-updater from running. This caused:
1. The MID server to delete some of its files to get ready for the upgrade.
1. The upgrader failed to run or failed to copy over the new files.
1. The MID server was missing files and couldn't start anymore.

This can also happen when something goes wrong with a manual upgrade. Not everyone gets it right, but once the damage has been done, it's done.

## Why you can't just reinstall
Sure, you could just reinstall the MID server from scratch. It feels like the damage can't be recovered from, so why even try?

However, you need to have some of the MID server's details before you can reinstall.
* The MID server's Windows username and password.
* The MID server's ServiceNow username and password.

> If you don't know these things, you cannot reinstall the MID server.

It's very unlikely that most ServiceNow admins and Windows engineers will have this information on-hand.

## How to recover
In a nutshell, what you'll want to do is to create a new MID server folder, and then copy over some key files from the existing MID server. This is enough to revive the existing MID server.

1. Rename the **"agent"** folder to **"agent_old"**. You're going to need this folder later on.
1. Set up a new **"agent"** folder with all of the MID server files. Either
 Copy these from another existing MID server, or 
 navigate to "MID Server - Downloads" in ServiceNow and download the MID server as a ZIP file, then unzip the **"agent"** folder alongside **"agent_old"**.
1. There should now be 2 folders: **"agent"** with a complete MID server and **"agent_old"** with the broken MID server.
1. Copy the below files and folders from **"agent_old"** to **"agent"**.
 /agent/config.xml
 /agent/conf/wrapper-override.conf
 /agent/keystore (if exists)
 /agent/security (if exists)
 /agent/jre/lib/security/cacerts
1. (Windows) Update the file permissions on the new **"agent"** folder so that the MID server's Windows user has full access to the entire folder.
1. Try to start the MID server again

If the MID server service starts and shows as "Up" in ServiceNow, you have successfully recovered the MID server after.

### What are those files
At it's heart, these are the files that differentiate one MID server from another.

**/agent/keystore/agent_keystore** or **/agent/security/agent_keystore**
This is a Java keystore file which stores important security and encryption keys.

Most importantly, it stores the key that the MID server uses to encrypt text, such as the MID server password in the 'config.xml'. If you lose this keystore, the MID server can't decrypt that password anymore.

**/agent/config.xml**
The MID server's configuration, including 
* the MID server's name
* the SN instance it belongs to
* the username and password it logs into the SN instance with

This file defines how the MID server works once it has started. If you lose this file, the MID server won't know who it is and where it belongs to.

**/agent/conf/wrapper-override.conf**
This file controlls how the Windows service works. Notably, the name of the Windows service.

If you lose this file, the MID server will keep trying to start the wrong Windows service, either starting the wrong one or throwing errors if the service doesn't exist.

## Links
* MID Server Down | A guide on how to restore the MID Server
 https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0535040
* How to manually restore or upgrade a MID Server after a failed auto-upgrade
 https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0713557