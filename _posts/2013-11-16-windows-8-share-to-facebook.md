---
title: Windows 8 Share to Facebook
date: 2013-11-16 00:23
categories: [Technical-Howto]
tags: [Facebook, Windows8, Windows8.1]
---

My favorite feature in Windows 8 has to be the Share Charm. I really think it's a great and unique idea.
If you don't know what the Share Charm is, do yourself a favor and read about it ASAP here:
[http://windows.microsoft.com/en-us/windows-8/charms-tutorial](http://windows.microsoft.com/en-us/windows-8/charms-tutorial)

One of the most trivial implementations to sharing is sharing to Facebook. To my big surprise, I found out that Windows 8 has no built in Share to Facebook option. The People app has Sharing capabilities, but it only lets you post to your profile. What about sharing to in a private message? What about posting to a group? In my personal experience these are far more common use cases.

I've been playing around with the idea of creating this Share to Facebook thing myself, but I always kept hoping that someday they'll just add it. The official Facebook app has a Share target, but it is just as useless as the People app share target.

So I gave up, I created my own Facebook share target. I'm just putting it here in case anyone else feels the same as I do about the built in Facebook sharing functionality. I'm not going to submit it to the store because I think it's a stupid app. I really hope Microsoft or Facebook will add this soon and this post will be obsolete.

Here it is in action:
![my FB sharer](/images/2013-11-16-windows-8-share-to-facebook_1.png)

The code is basically just this:

```javascript
(function () {
    'use strict';

    var app = WinJS.Application;
    var share;
    var fbShareUrlTemplate = 'https://www.facebook.com/sharer/sharer.php?u=';

    app.onready = function (args) {
        theWebView.addEventListener(MSWebViewNavigationStarting, navigationStarting);
    };

    app.onactivated = function (args) {
        if (args.detail.kind === Windows.ApplicationModel.Activation.ActivationKind.shareTarget) {
            share = args.detail.shareOperation;

            if (share.data.contains(Windows.ApplicationModel.DataTransfer.StandardDataFormats.uri)) {
                share.data.getUriAsync().then(function (uri) {
                    if (uri != null) {
                        var urlToNavigate = fbShareUrlTemplate + uri.absoluteUri;
                        theWebView.navigate(urlToNavigate);
                    }
                });
            }
        }
    };

    function navigationStarting(args)
    {
        var url = args.uri;
        if (url.substr(0, fbShareUrlTemplate.length) !== fbShareUrlTemplate) { // if the url doesn't start with fbShareUrlTemplate
            args.preventDefault();
            share.reportCompleted();
        }
    }

    app.start();
})();
```

A couple of things to note:
- It uses Facebook's deprecated sharer.php.
- It can only share urls, like from IE.
- It is poorly written and tested.

Yes, I am lazy. But it's serves me well, and I'm happy with it. And it just proves how simple it is to implement, so Microsoft \ Facebook – Please do!
