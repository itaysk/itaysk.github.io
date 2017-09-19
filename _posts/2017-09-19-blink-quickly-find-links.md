---
title: Blink - quickly find links
date: 2017-09-19 22:22
categories: [Technical-Howto]
tags: [azure-search, angular, node-js, azure-functions]
---

I've built a collaborative bookmarking tool for my team at work. We've been using it for a long time now, and I figured why not OSS it. Find it on GitHub: [https://github.com/itaysk/blink](https://github.com/itaysk/blink)

![Blink search for stocks](/images/2017-09-19-blink-quickly-find-links_1.png)

## Problem

Microsoft is a huge organization with a vast intranet of knowledge, tools, and content. No one can remember how to get to all of this stuff. Generally people are devided into two categories:

- Those who email the team, asking who knows how to do X.
- Those who have gained some milage with the team and have acumulated a decent inbox over time, so they search their inbox to find how to do X.

Eventually people will organize the most useful links for themselves in their browser's bookmarks.

I hated that too much is shared over unstructured email, and that everyone keeps basically the same bookmarks for temselves. I was probably driven by the fact that since I'm the veteran in the team, I was the one usually doing the inbox digging to answer people's emails.

## Solution

I created Blink as a shared bookmaring service. You should think about it the same way as your browser's bookmarks, just shared with the team.

- Looking for the travel request form? Search Blink for 'travel'
- Looking for the Azure cunsumption report for the customer you work with? Search Blink for 'azure consumption'
- Looking for your HR profile information page? Search Blink for 'HR'
- Found an interesting page on the roadmap of some product? Submit it to Blink and tag it with relevant keywords so that if you or your colleague will ever need it sometime, you'd recall you once saw this useful page, and use Blink to get to it again.

## Technology

As you might have guessed, it's built on Azure, for Azure (We also sometimes use it as a demo for Azure Search or Azure Functions).  
I tried to keep it simple, and optimize for cost over time, and searching experience. As a consequence adding links is slow.  
Front end is Angular 4. I'm not a Front End Developer. I suck at UI. Contributions are welcome :)

## More

On GitHub: [https://github.com/itaysk/blink](https://github.com/itaysk/blink)

