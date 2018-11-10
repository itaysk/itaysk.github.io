---
title: Determine Who Created Resources in Azure
date: 2017-06-25 13:42
categories: [Technical-Howto]
tags: [azure-functions, governance, oms, log-analytics]
---

When managing an Azure subscription a common request it telling who created a resource or a resource group. Unfortunately Azure doesn't make this information easily accessible (in fact, I think it doesn't event hold this data), so in this post I'll suggest several workarounds to this request:

## Approach 1: Activity Log
Every action you make in Azure (regardless of who you did it, Portal, API, CLI) is audited and registered in Azure Activity Log.  
Activity Log events does include information about the user who initiated the request. We could query the log for the history of the resource we are interested in, specifically we are interested in the very first action performed (that would be the action to create that resource), and look at the user who submitted that action.  

To view the Activity Log, open Azure Monitor, and click on Activity Log in the menu.  
Filter by the Resource Group and Resource you want to investigate, and set the date range to the last 90 days.

![Activity Log](/images/2017-06-25-determine-who-created-resources-in-azure_1.png)

See the caller in the "Event Initiated By" column? That's what we were looking for.

There's one important limitation with this approach: the Activity Log has a fixed retention period of 90 days, meaning you can't look back more than 90 days in the past (hence the next approach).

## Approach 2: Log Analytics (OMS)
The Activity Log is limited to last 90 days, but we can continously export the log into an infinite Log Analytics account (also part of OMS).  
First you need create a Log Analytics account, and then configure Azure to forward all activity logs to the Log Analytics account. This is an easy integration: [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-activity](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-activity).  

Now we can use the Log Analytics search to find the first action on the resource, and looks at it's initiator.

![Finding caller in Log Analytics](/images/2017-06-25-determine-who-created-resources-in-azure_2.png)

Here is an example query:

{% gist itaysk/25c846b1461fa6e81399601b2ca2fa70 file-query-kusto %}

(The post originally published with the following query written in the legacy OMS query language. I'm keeping it here for reference:)

{% gist itaysk/25c846b1461fa6e81399601b2ca2fa70 file-legacy-query-oms %}

## Approach 3: Automatic tagging
If you need the ownership data more accessible, another approach will be to capture this information in tags that are easily accessible for every resource.  
We can use Alerts to hook into Activity Log events, and implement a web-hook handler that tags resources with the user information.
I have made a Proof-Of-Concept project that uses Azure Functions to implement this flow. 

Read more info about this scenario in GitHub, here:
[Azure-TagOwner](https://github.com/itaysk/azure-tagowner).