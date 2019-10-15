---
title: kubectl-neat cleans up Kubernetes YAMLs
date: 2019-10-15 21:50
categories: [Technical-Howto]
tags: [kubernetes, go]
---

> TL;DR: kubectl-neat is a kubectl plugin that cleans up Kuberntes YAML and JSON output to make it readable by removing default values, runtime information, and other internal fields. This post is about the story and experience behind creating it. If your are just interested in getting started, check out the GitHub page: [https://github.com/itaysk/kubeclt-neat](https://github.com/itaysk/kubectl-neat).

## Background

You know how sometimes you create a *simple* Pod, like 10 lines of YAML, and then a second later you try to `kubectl get -o yaml` that same little Pod, and you get back like 100 lines of YAML that you didn't even know about? What happened here is that as Kubernetes processed your Pod request (or for that matter any other resource) it added a whole bunch of internal system information. This includes:

- Metadata such as creation timestamp, or some internal IDs
- Fill in for missing attributes with default values
- Additional system attributes created by admission controllers, such as service account token
- Status information

kubectl-neat is a little tool that I created to scratch that itch. It cleans up those bloated YAMLs and gives you back a readable output.

## In the beginning there was jq

As many good things, it started with some `jq` magic. I have to be honest - I LOVE jq, and I think it's awesome. jq allowed me to express some basic "static" rules to clean up known garbage. By static, I mean things that I know that that I always want to remove, regardless of the actual document I got. For example, removing `pod.status`, or relieving `pod.metadata` from creation timestamp and history tracking and internal ids and whatnot. It was even fine for removing things like the auto-mounted service account token volume, which I didn't specify in my YAML but some admission controller has added.  
This jq magic was wrapped by some bash to make it an actual plugin. and it was nice, but wasn't enough.

## From static jq to dynamic defaults

It turns out that a common pod includes a lot of "garbage" fields that I really didn't care about, and most of this came from Kubernetes filling in default values *for every single field in the spec*. If you created a resource with just 2 fields defined, but the resource definition for that kind has 100 fields, Kubernetes will likely fill in those other 98 fields for you, thank you very much.
So next step is to detect and remove default values. In my research I discovered some bad news and some good news:

### The bad news

I hoped to find some schema definition that included the schema of known resources, and their default values, but I found that there's actually business logic, in the Kubernetes source, for how to determine default values. When you think about it, it makes sense because some of the default values may depend on the values of other fields. For example, the default value for `imagePullPolicy` depends on the image tag: If the tag is `latest` it `Always` and if it something else it's `IfNotPresent`. Here's the [source code for that](https://sourcegraph.com/github.com/kubernetes/kubernetes/-/blob/pkg/apis/core/v1/defaults.go#L80):

```go
	if tag == "latest" {
		obj.ImagePullPolicy = v1.PullAlways
	} else {
		obj.ImagePullPolicy = v1.PullIfNotPresent
	}
```

That's bad news because it's going to be harder than I thought to determine which values are default and should be removed.

### The good news

The good news is that Kubernetes has nicely factored codebase , which means there's a generic way to invoke the defaulting logic of any arbitrary Kubernetes object.

To be more specific: [`ObjectDefaulter`](https://sourcegraph.com/github.com/kubernetes/apimachinery/-/blob/pkg/runtime/interfaces.go#L192) defines an interface for something that can fill in default values, which `Scheme` [is implementing](https://sourcegraph.com/github.com/kubernetes/apimachinery/-/blob/pkg/runtime/scheme.go#L389) by generically loading the [`defaulterFuncs`](https://sourcegraph.com/github.com/kubernetes/apimachinery/-/blob/pkg/runtime/scheme.go#L70) that each [api package declares](https://sourcegraph.com/github.com/kubernetes/kubernetes/-/blob/pkg/apis/core/v1/zz_generated.defaults.go#L31).

Without going into the details of Kubernetes API Machinery, the good news is that we can write some Go code to invoke the same defaulting logic for us, and use it to "dynamically" detect default values and remove them. This time it's dynamic because it does take into account the entire state of the object, and predicts the correct defaults as closely as possible to what Kubernetes would do.

## POC phase

Indeed I created a little tool that predicts default values and asserts if a given value is default. I then meshed that into the bash+jq mix that I had, to create an actually functioning plugin that removes most of the "garbage" that I was aiming to remove. I really enjoyed this as an exercise in writing some real bash, including tests and everything, and sharpening my makefile and jq skills. You can browse the repo at the [`v0.1.1` tag](https://github.com/itaysk/kubectl-neat/tree/v0.1.1) to see how that looked like.

But you know how this works, everything bash is doomed to be re-written in a "real language" when it grows. Jokes aside, it was clear to me that I'll eventually re-write this in Go, as some of the code was already Go (the defaulting part), and if I wanted more people using it, I had to make distribution dead simple.

## Go re-write

I have to say that coming from other languages, Go really lacks when it comes to JSON manipulation. I started by going "the Go way", which means: using the standard `encoding/json` package, using typed objects whenever possible (what a rabbit hole that was, diving into api-machinery), and many many empty interfaces... This was a nightmare. I don't know if it's because I'm not a super experienced Go programmer, or I'm not a "Go person", or maybe "the Go way" actually sucks, but it was so unwieldy that I had to take a break for a few weeks. When I got back at it I decided to use the [`gjson`](https://github.com/tidwall/gjson) library. It still wasn't as straightforward as I hoped it would be, but it worked out and I only had to write one recursive function in the whole process :P

## Next steps

I feel fine with calling this `v1.0.0`. I submitted this to the official krew index, and it now available with a simple `kubectl krew install neat`. My next steps are better build and test automation, and adding windows support. Additionally, there are some improvements to the code to be done around JSON manipulation, and the defaulting logic. Finally I'm entertaining the idea of using this as a linting tool. If you are interested in any of these, or have any other feedback/questions, feel free to contact via GitHub issues or Twitter.