---
title: kube-conditioner - Custom Pod Conditions For Everyone
date: 2019-03-27 19:45
categories: [Technical-Howto]
tags: [kubernetes, prometheus]
---

> TL;DR: 
> kube-conditioner is a Kubernetes controller that allows you to define custom Kubernetes Pod conditions based on your own logic. Check it out here: [https://github.com/itaysk/kube-conditioner](https://github.com/itaysk/kube-conditioner).

## Story time

Recently I was leading an effort to build a [canary release](http://blog.itaysk.com/2017/11/20/deployment-strategies-defined#canary-deployment) management tool for Kubernetes applications.  
Consider a canary release pipeline, where one of the phases would need to verify the status of the experiment and decide how to act accordingly (this is a trivial part of canary release practices). A basic example might be that we have a new version of the shopping cart service of our app - we want to release it to a small group of users and verify that the error rate they experience is below a certain threshold. How will my canary management tool know if the error rate for the canary version was good or not? I was looking for a way to capture this information in an actionable and Kubernetes friendly way.  
I though about how every resource in Kubernetes has a [Status](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) section that presents information about the state of the resource, and I began by trying to add my canary status information to the status of Pod resources, knowing that the canary management tool will be able to easily fetch this information. What I found is something slightly different but even more useful - Pod Conditions.

## Pod conditions

Pod conditions are standard part of the Pod status. They are described as:

> Conditions represent the latest available observations of an object's state. They are an extension mechanism intended to be used when the details of an observation are not a priori known or would not apply to all instances of a given Kind
[source](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)

Some Kubernetes resources already make use of conditions, most notable - Pods. If we look at [pod conditions](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions), we see some basic types of conditions: `PodScheduled`, `Ready`, `Initialized`, `Unschedulable`, `ContainersReady`. These are some conditions that Kubernetes maintains by default. Could we add our own condition here?

### Manipulating pod conditions

You can quite easily add your own custom pod condition using the Kubernetes API. For example, here's how to add/update a custom condition via the Kubernetes REST API:

{% gist 572d9b46a6dd4284970670bf2ca3acfb update-rest-strategic.http %}

## Custom controller

In order to maintain an up to date status of our condition we need some process to continuously monitor and refresh the condition. This calls for a [controller](https://kubernetes.io/docs/concepts/extend-kubernetes/extend-cluster/#extension-patterns) to own our custom condition and make sure it's always up to date. So I've created a generic controller to do exactly that!  
kube-conditioner allows you to define Kubernetes objects called `PodConditions`. In this CRD you specify a KPI (for example look at Prometheus for error rate for this app and make sure it's within a threshold), a group of Pods to update (by label selector), and an interval for how often to evaluate the condition. kube-conditioner will create and maintain this custom condition for you.

## Use cases

Now that we have our custom Pod conditions, how can we use them? 

### In scripts

First low hanging fruit - they show up in the pod status, so by `kubectl describe pods ${yourpod}` you can get a very quick indicator if your condition is in good shape or not. This can be easily automated and incorporated this into your day to day operations, in scripts or in CI/CD pipelines. Here's an example how to get the status of `mycondition` condition of `mypod` pod:

{% gist 572d9b46a6dd4284970670bf2ca3acfb get-kubectl-jq.sh %}

### Pod Readiness Gate

A recent addition to Kubernetes allows us to pick some Pod conditions, and make them a criteria for Pod readiness state, so the pod will be "Ready" only if those condition is true - This is very very cool! It means that Kubernetes can take unhealthy pods **by your standards** out of service automatically.  
More on readiness gate here: [https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate) .

### Automatic remediation

Consider a more sophisticated use case where some other tool subscribes to changes in pod conditions and reacts in an automated way. This is an extremely appealing use case for many operational aspects.

## Additional reading

- More on conditions https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties
- Pod readiness gate original spec (KEP) https://github.com/kubernetes/enhancements/blob/master/keps/sig-network/0007-pod-ready%2B%2B.md
- node-problem-detector is the only other project I found to leverage conditions (but for nodes) https://github.com/kubernetes/node-problem-detector


