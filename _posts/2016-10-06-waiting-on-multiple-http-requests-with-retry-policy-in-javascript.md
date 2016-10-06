---
title: Waiting on multiple HTTP requests with retry policy in JavaScript
date: 2016-10-06 13:50
categories: [Techical-Howto]
tags: [JavaScript, node.js, promise]
---

For a node.js task I was working on I needed to do the following:

* Submit many (hunderds) of http requests to the same API endpoint.
* Wait for all of them to finish and report their status.
* Since I am hammering this API, throttling is expected so - Handle API throttling and transient errors using a retry mechanism.

I was hoping to find many code samples and solutions for this, but unfortunately couldn't't find any clear, simple samples.

Here's what I did (explanation after the code):

{% gist itaysk/3c3a5c36fae31e464802b61f1239bb2c %}

First step -

We need to create an array of requests, and wait on them. I used [request-promise](https://github.com/request/request-promise) to promisify request, and [bluebirdjs](http://bluebirdjs.com) to wait on all the promises.


Second step -

The problem with `Promise.all()` is that it will fail as soon as any of the promises fails. From bluebird's docs:

>  If any promise in the array rejects, the returned promise is rejected with the rejection reason.

Therefore we need to transform each individual request promise to one that always succeed. we do that using bluebird's `reflect()`.

Third step -

Now we need to add the retry mechanism, I used [bluebird-retry](https://github.com/demmer/bluebird-retry) to wrap the request in a retry-able promise. 


To summarize, the relationship between the 3 steps is as follows: `reflect(retry(request))`.