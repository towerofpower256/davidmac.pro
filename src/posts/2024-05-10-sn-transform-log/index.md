---
title: Why your ServiceNow transform logging doesn't work
description: You feel like you're going mad. You've used log.info in your transform scripts, but there's no messages getting logged. Here's why it does that and how to change it.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-05-10
tags:
- ServiceNow
eleventyExcludeFromCollections: false
---

## The problem
So you're running a transform to import some data into ServiceNow, but something has gone wrong. For debugging, you find or add some `log.info()` to the transform script, but when you run the transform you find that nothing has been logged to the transform log.

[![Transform script with logging](transform-script-with-logging.png)](transform-script-with-logging.png)

[![Log messages missing](transform-log-missing-info.png)](transform-log-missing-info.png)

## The answer
Starting around the San Diego release, using `log.info()` won't actually log messages to the transform log. It will log `log.warn()` and `log.error()`, but not `log.info()`.

To change this, set the system property **glide.importlog.log_to_table** to **true**, or create it if it's not there already.

[![System property to save info logs](system-property.png)](system-property.png)

[![Transform log with info](transform-log-info.png)](transform-log-info.png)

## Fun fact
What's actually going on here is that `log.info()` **does** log, but the log messages aren't saved to the database under the transform log anymore.

However, they still get logged to the node's log file. You can see the `log.info()` messages if you browse or download the node's log file.

This isn't useful, but still cool to know.

[![Transform logs in node logs](node-logs.png)](node-logs.png)

## Links
* SN Docs - Import sets properties
 https://docs.servicenow.com/bundle/tokyo-platform-administration/page/administer/import-sets/reference/r_ImportSetsProperties.html
* SN Community - Import logs are not created when you import something using the transform maps from SanDiego Version
 https://www.servicenow.com/community/cmdb-forum/import-logs-are-not-created-when-you-import-something-using-the/m-p/2393960