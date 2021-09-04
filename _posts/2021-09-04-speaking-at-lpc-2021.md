---
title: I'll be speaking Linux Plumbers Conference 2021
date: 2021-09-04 16:45:00
categories: [Events]
tags: []
---

Towards truly portable eBPF
As eBPF is getting more popular and mainstream, one of the challenges of making it accessible to more users is how to distribute eBPF powered applications. Unlike simpler applications which involves shipping a binary or a container image, with eBPF we usually need to compile the program for the target kernel. This is a hurdle in adoption by both users and vendors. The CO-RE (Compile Once - Run Everywhere) initiative improved this by introducing a way to ship a compiled artifact, which will work on any supporting distribution. But what is a supporting distribution and what about unsupported distributions? How can we make eBPF CO-RE widely usable in the real world of enterprise users? In this talk we will answer these questions by introducing CO-RE and BTF mechanics, and how to leverage them in a concrete scenario in our project Tracee.

https://linuxplumbersconf.org/event/11/contributions/948/