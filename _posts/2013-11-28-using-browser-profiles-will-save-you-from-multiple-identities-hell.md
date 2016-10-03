---
title: Using Browser Profiles Will Save You From Multiple Identities Hell
date: 2013-11-28 16:12
categories: [Technical-Howto]
tags: [Chrome, FireFox, Internet Explorer, Office365]
---

I have like 10 Office 365 identities: I work for Microsoft, so obviously we use Office 365 internally; I also help some customers maintain their Office 365 tenants; I also have one or two Office 365 account for demos, and for each of those I have several sample user accounts to demonstrate collaboration scenarios.
It's a real pain to work with all these alternating identities in the same browser. And it doesn't stop there, because for each Office 365 account, I also have an associated Yammer account, and for some I also have associated Windows Azure accounts, and you can see where I am going with this.

First lets agree that signing in and out of services sucks. You really don't want to do this. Other solutions like separating different kinds of use cases into different browser types is lame, and private browsing (incognito) is inconvenient, and not sufficient solution.

So the requirement is essentially to maintain multiple personas, each with it's own set of credentials an settings. This actually is available today in most browsers but I found that the best implementation of this concept was done in Chrome (not surprisingly).

## End Result
I have a group in the start screen with shortcuts for each persona.

![end result](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_1.png)

I can launch multiple browsers with different identities.

![identity 1](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_2.png)

![identity 2](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_3.png)

For each user I can set different bookmarks (i.e one is admin, other is user).

I can choose to pin my primary personal user to taskbar, but not test users'.

![taskbar](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_5.png)

Best of all I don't need to fear from checking "remember my password"!

## Chrome
Chrome makes it very easy to setup profiles. Just go to the Users section under the Settings screen. There you can add users as you wish.

![chrome profiles](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_6.png)

The UI let's you create a shortcut for the newly created user instance. You can launch chrome under a specific profile like this:
`chrome.exe --profile-directory="Profile Name"`
I really liked that chrome lets you pick a different icon for each profile so you can tell them apart when you have several opened at the same time.
You can also switch profiles at runtime, a feature that I find useless and confusing actually.

## Firefox
Firefox has profiles support but it's not as easy as Chrome's.

first you need to open the separate profiles manager (FF needs to be closed before running this):
`firefox.exe –p`

![firefox profiles](/images/2013-11-28-using-browser-profiles-will-save-you-from-multiple-identities-hell_7.png)

You then run different profile instances using
`firefox.exe –no-remote –p profile name`

I couldn't find a way to set different taskbar icons for each profile, which I this is an important feature when working with multiple identiites at the same time.
There's also no built in profile selector, but there's an extension available, and I don't use this feature anyway.

## IE
IE doesn't have the concept of profiles. You can use `File->New session` to open a new window that is isolated in terms of authentication, but it will share your bookmarks and other settings.

## Summary
These days everything is in the cloud, and everything is "as a service". A side effect of this approach is excessive use of identities. This is true for work, home, mobile, everywhere. If there are online services for which you have multiple accounts, you should really check out profiles and sandboxing capabilities of modern browsers, especially Chrome.
