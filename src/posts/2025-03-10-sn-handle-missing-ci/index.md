---
title: How to handle a missing CI in ServiceNow
description: Let's say you're raising a new Incident and the "Configuration Item" field is mandatory, but the CI you need isn't in the CMDB. You can't proceed without it, so you're stuck. What do you do? I asked a few colleagues what strategies they recommend, here's what they said.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2025-03-10
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
---

## The poll 
I did a poll on LinkedIn, asking people what their strategy was for a missing CI, here's what I got: 

https://www.linkedin.com/posts/davidmcdonaldbrisbane_servicenow-servicenowadmin-servicenowarchitect-activity-7267007777565270018-_TQc  

* A **Catalog Item** called **"Report a Missing CI"** to self-service report a missing CI

* A **placeholder CI** called "MISSING CI", and a data sanity process to watch out for tasks using this CI, and resolving the as required. 

* ITIL fulfillers have the ability to **create CI's in the CMDB as they need to**. (this one didn't come up much, I don't think it was popular). 

* Someone that you bug with an email or Teams message whenever a CI is missing. 

The focus is on a missing **technical** CI (e.g. computer, database, router, etc), and not a **service** or **application**. If your CMDB is missing one of those, you need service owners to get involved. Let's just focus on a missing **technical** CI. 

Here's what they had to say.

> The question is…is there a CMDB admin that is responsible for ensuring CIs are populated in the CMDB and being maintained? If so, that person is responsible for tracking down why the CI isn’t in the CMDB and determining the best course of action. It’s called Governance! 

This stresses that proper governance should make it impossible for there to be any missing CI's.  

**I disagree**: although governance is important and should make it very unlikely for a CI to be missing, there's still a chance for a CI to be missing for some unexpected reason even if governance is near-perfect. 

>  It's a little thing called PROCESS. Process, process, process. Did the person who commissioned the CI follow documented process and submit a Request or CHG task to notify Config Mgmt of the new CI? That way, the business is being 'proactive' instead of 'reactive' to ensure the CI is populated into the CMDB via discovery or other means. Config Mgmt can't be expected to know about CI's if they're not informed of them.  
>  
> A generic CI to allow CHG's to be submitted and completed when/if a CI is not found will allow business continuity in the meantime. 

Similar to before, this person stresses that proper governance should make it highly unlikely that a valid CI can be missing from the CMDB. 

It does note that they'll still use the generic "MISSING CI" placeholder to capture & identify cases where CI's are missing. 

> We have a No CI Found to allow the L1 to escalate to the L2 group in the CI, which is the unsupported software queue. I think investigate it & help the user best I can. Have rooted most things out over the years. 
>  
> Does lead me to create CIs that aren't supported in ServiceNow, but they all route to my queue or I put in the CI that it's not supported & here's what to do via the support information field. 

Tt sounds like the placeholder "MISSING CI" strategy is used. **This is sounding like a rather popular option.** I like the process that the placeholder "MISSING CI" routes jobs and changes to a group called "Unsupported Software", it's a great way to make it visible to those that need to know. 

> I have experienced similar kind of issue in of my projects where the CI went missing. We used Missing CI Catalog Item to add it manually. 

I like this idea of being able to self-service report missing CIs. It allows citizen users of the platform to help identify these gaps in the CMDB. 

## Conclusion 
It sounds like all methods are feasible. You could do one or both, and easily unclog the process: 

* A "Report a missing CI" catalog item. 
* A "MISSING CI" placeholder CI that is very visible to Config Management teams to indicate that a CI is missing, but still doesn't hold-up ITIL processes (e.g. Incident management for that broken machine). 

Nobody recommended that ITIL users should be able to create new CIs as they need them.

However, there should still be a focus on good **governance** for Configuration Management to help prevent a missing CI in the first place. 