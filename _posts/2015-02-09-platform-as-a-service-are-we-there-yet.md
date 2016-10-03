---
title: Platform as a Service – Are we there yet?
date: 2015-02-09 10:04
categories: [Thoughts-Opinions]
tags: [Azure, Cloud, PaaS]
---

The following topic has been occupying my mind ever since the time I was first learning about Azure and Azure Cloud Services (aka Web and Worker Roles). At that time I was only beginning to understand "the cloud" but was already familiar with concepts such as IaaS and PaaS. Particularly, I was very much into the concept of PaaS. 
Also at that time Azure was all about Cloud Services. There were no VMs or Websites or any of the other compute solution that are available today. So there I was, all in for the new PaaS stuff, excited to see Microsoft's take on it, and that's when it hit me.

You see, hearing about PaaS in a theoretical (almost idealistic) manner, I was expecting an amorphic, boundless, elastic cloud, that you can just throw code at, let it do its thing, and you just get the result out. That's it. No infrastructure, no setup, no planning ahead, no fixed costs, and no commitments. I don't want to know the details such as what OS or stack this cloud is running. For all I'm concerned it's using hamsters to turn wheels that power a Turing machine to evaluate my code. As long as I get my response on time – I just don't care.
Imagine then how confused I was when one of the first things I learned about Azure (PaaS) is that you have to choose VM size and number of instances that will run your code.

![paas scaling](/images/2015-02-09-platform-as-a-service-are-we-there-yet_1.png)

What I had in mind back then (and what I still wish for today) was a pure PaaS solution. One that I think of like any utility service. From time to time I give talks about the cloud, and I like to use the analogy of electricity and cloud computing. When you consume electricity you don't think about how many generators are operating your neighborhood, or how much electricity each generator produces, you just stick your appliance in the wall and it just works. A true "as a service" experience. Likewise, when I put up a website up in the cloud, I don't want to think about how many servers are operating it, or how powerful each one of them has to be. I just want to give you my code let you run it. Charge me for what I used, just like electricity.

Since that time where I started to learn about Azure Cloud Services, Azure introduced more PaaS solutions, like Azure Websites. I really like Azure Websites, and I think it's a step towards my ideal PaaS solution. For example, I actually like it that I can't RDP into the machine, and that you don't have to deal with the application stack. But still I have to choose VM sizes and instance count.
Amazon introduced lately AWS Lambda, which in my opinion is another step forward. I like that you are billed by the number of requests you make and the time it took to process them. That's a very "as a service" approach. You do have to choose the size of the compute node that will run you requests, but not the number of instances. 

I really hope to see this trend evolve. I hope that eventually we will get to a point where we can offer a true, pure platform as a service.
I do see the challenge though. Especially if you use current technologies to implement such platform.
There are other PaaS components today that does manage to offer this kind of experience. Storage, Queues and Media services are just some examples for services where this model is successful. The storage service is just that thing you throw files at, and it just works. You never have to think about how many computers operate it or what kind they are. 

What's the difference then? Why can it work for some cases and not the others? What's common to all those true PaaS solutions is that they are all very well defined services, very well scoped, and answer a very specific need. All users of these services use them the exact same way. A push to a queue is a push to a queue, the server side is very predictable. When you have a service like this, you can start standardizing the infrastructure and operations that run it. You can confidently estimate how much resources you require per request, and you can base your billing and SLA models on that instead of raw resources consumption. In the media services example, the algorithm to transcode from A to B is always the same algorithm. The only variable is the length and maybe the resolution of the movie. So you can take those parameters and make them your billing and SLA units.

That is different from general compute applications. Are all web requests the same? Of course not. A request for a simple marketing campaign website is very different from a request for an image processing service, so it's very hard to fit them both into a unified pay by request model. You could start charging by the time it take to process the request, sounds fair, but what about latency requirements? How fast should your code run, is a question only you can answer, and something that is tightly coupled with your specific code. I can go on with this but I think you get the point – General compute is too general to be easily turned to "as a service".

There are many issues involved with making my dream Platform as a Service, but I think it may be possible. Specialized services like those that I have mentioned and others are already providing this experience, and people love that. Technology companies (such as Microsoft) are pushing Platform as a Service and are making them the focus of innovation in today's technology, and businesses are seeing the value in it. We are ready for a truly elastic, self-managed PaaS solution, and I hope we will get to see one soon enough.
