---
title: Hacker News Visual Rank extension
date: 2016-10-24 12:18 
categories: [Technical-Howto]
tags: [chrome, edge, firefox, github]
---

[Hacker News](https://news.ycombinator.com) is one of my favorite sites, every idle minute I have is probably spent there. For those who don't know, It's a tech news sharing site that puts users first - users submit items, users vote on interesting items, and users discuss items (similar to reddit).

I noticed that whenever I read the front page, when I skim over items, I always take note of how many points and comments each have. It's became an automated pattern for me to see what others though of that item.
The way items are structured on the page makes it annoying to read if you care about the number of points and comments - Your eyes will jump from side to side, see for yourself

![HN item](/images/2016-10-24-hacker-news-visual-rank-extension_1.png)

So I decided to build a browser extension to help me read HN the way I like. What is does is to change the font size of items according to the number of points and comments they have. So items with many point and comments will be large, and items less popular will be small.
This is *not* instead of HN's native ranking algorithm. The site clearly has an internal logic to sort items based on those parameters, and I respect and keep that. While keeping items sorted in according to their HN rank, I add another dimention by using font size.

Here is the result:

![screenshot](/images/2016-10-24-hacker-news-visual-rank-extension_2.png)

## How to get it

- [Source code](https://github.com/itaysk/Hacker-News-Visual-Rank)
- [Chrome](https://chrome.google.com/webstore/detail/hacker-news-visual-rank/hnbdiaedemlcfnjpdgadhhhdmmhhlncm)
- [Firefox](https://addons.mozilla.org/addon/hacker-news-visual-rank/)
- For Edge you can get the source and load the extention locally. Edge extension support is not open for developer submissions yet.