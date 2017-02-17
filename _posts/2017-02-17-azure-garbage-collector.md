---
title: Azure Garbage Collector
date: 2017-02-17 11:55
categories: [Technical-Howto]
tags: [azure-functions, powershell, logic-apps]
---

At work, we have a shared Azure subscription that we use for learning, demos, experiments, etc. This account can grow very fast on expenses, and it's easy to lose track of what's going on there because there are many different people using it for different projects.  
So I have decided to create a tool that helps us keep order in that subscription, I call it 'Azure Garbage Collector'.  
I have mentioned this to some of the customers I work with and they found it useful as well (mostly for dev\test scenarios), so I decided to publish it.  
The complete system is [on my GitHub](https://github.com/itaysk/AzureGC) with detailed documentation. Feel free to use it, tweak it, and distribute it.  
If you have requests or feature suggestions - please open an issue on GitHub. Pull requests are also welcome!

Find it on: [https://github.com/itaysk/AzureGC](https://github.com/itaysk/AzureGC)


![system overview](/images/2017-02-17-azure-garbage-collector_1.png)