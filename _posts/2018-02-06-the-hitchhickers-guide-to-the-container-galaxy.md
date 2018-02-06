---
title: The Hitchhiker's Guide to the Container Galaxy
date: 2018-02-6 10:00
categories: [Technical-Howto]
tags: [kubernetes, docker, linux]
---

The container landscape is evolving, and we are witnessing it's evolution as new projects and ideas emerge, standards are being formed and new tools are developed. If you are closely watching this space, you might be familiar with the plot, and everything will make sense to you; but for a newcomer, it might be overwhelming to encounter the plethora of terms, names, and projects that came to be. So in this post I'll try cover the important terms and explain their origin and motivation.
  
> TL;DR: none of this matters to you as a Docker/Kubernetes user! It's mostly about internal infrastructure plumbing and refactoring of the code base. Yet it's an interesting story to hear. 

I will break the story into to two timelines, one for the container ecosystem, and the other for the Kubernetes ecosystem.  
Spoiler alert: This is what we'll get at the end

![timeline](/images/2018-02-06-the-hitchhickers-guide-to-the-container-galaxy_1.png)

## Containers

Linux introduced kernel features such as cgroups and namespaces as early as 2002, and have gradually added more in the following years. These features are the building blocks of what's referred to today as 'containers', but using them required deep knowledge and understanding, and also there was little user-space programs that exposed those features.

One of the [early projects][1] to package these features as a user facing tool was the 'Linux Containers' project, or 'LXC'. LXC's goal was to replace hardware virtualization (VM) with more lightweight OS virtualization by using (then) new kernel features. LXC was better known as a way to emulate an entire system, unlike today's containers which are more application scoped (although it was capable of both system and application virtualization). This direction was emphasised by LXC's evolution into 'LXD'.

In 2013 Docker came and revolutionized the industry by making containers mainstream. When they lunched, they used LXC as the underlying container runtime.

In [March 2014][2], Docker refactored their product and extracted the container runtime from the docker tool. They also made this layer pluggable, so users could swap execution drivers and still use the familiar docker command line tool. In the same breath they introduced a new container runtime to replace LXC. This new container runtime was called 'libcontainer'. It was essentially a go library for working with containers, which was being used by the docker command line tool.

In [late 2014][3], CoreOS, originally focused on a Docker native Linux distribution, launched an alternative to Docker, called 'rkt' (initially called 'Rocket', eventually [donated to CNCF][4]). rkt had introduced some novel features and concepts (the pod), but the most important aspect of rkt was that it pitched an open standard for containers (which Docker lacked). The standard was called Application Container, or 'appc' for short, and included a spec for the container image format called 'Application Container Image', or 'ACI' for short. rkt was an implementation of the appc standard.

Then, a monumental event in the history of containers happened, the Open Container Initiative (OCI) formed in [2015][5] (originally under the name 'OCP'). This new Linux Foundation project's mission to standardize the container industry. It was founded by all the major industry players, including Docker, CoreOS, Google, Red Hat. The OCI has a spec for container runtime and container image, aptly named: 'OCI runtime-spec' and 'OCI image-spec'. 

BTW, CoreOS was a founding member but initially [appc and the OCI spec co-existed][6], [until they eventually merged][7].

The OCI has a reference implementation for the runtime-spec that is called 'runC'. It was [donated to the OCI by Docker][8].

In [December 2016][9] Docker spawned out another layer of their product - 'containerd' ([eventually contributed to CNCF][10]). containerd is an [interface for controlling and operating runC][11], exposing higher level container lifecycle operations. containerd has smaller scope then docker, and is more focused on being embeddable.  
Today [docker is using containerd, which is using runC][12]:
![docker architecture](/images/2018-02-06-the-hitchhickers-guide-to-the-container-galaxy_2.png)

In [April 2017][13] Docker announced 'Moby'. Some baffling explanation for what it is was provided, but the simple explanation is it's docker rebranded, and the popular interpretation is it's a marketing move for distinguishing the company 'Docker' from the open source project now called 'Moby'. *People still use Docker to describe pretty much everything in the container ecosystem, including the Moby project.*

[1]:https://lists.linux-foundation.org/pipermail/containers/2008-September/013237.html

[2]:https://blog.docker.com/2014/03/docker-0-9-introducing-execution-drivers-and-libcontainer/

[3]:https://coreos.com/blog/rocket.html
[4]:https://coreos.com/blog/rkt-container-runtime-to-the-cncf.html

[5]:https://web.archive.org/web/20160313090034/https://www.opencontainers.org/news/announcement/2015/12/%E2%80%8Bopen-container-initiative-establishes-technical-governance-announces-new

[6]:https://coreos.com/blog/making-sense-of-standards.html
[7]:https://coreos.com/blog/oci-image-specification.html

[8]:https://blog.docker.com/2015/06/runc/

[9]:https://blog.docker.com/2016/12/introducing-containerd/
[10]:https://blog.docker.com/2017/03/docker-donates-containerd-to-cncf/
[11]:https://blog.docker.com/2015/12/containerd-daemon-to-control-runc/
[12]:https://blog.docker.com/2016/04/docker-engine-1-11-runc/

[13]:https://blog.docker.com/2017/04/introducing-the-moby-project/

## Kubernetes

Kubernetes was released in [2014][14] by Google, as an open source container orchestration platform, that was also donated to the CNCF.

Initially, Kubernetes was just a way to run Docker containers in a cluster, so it worked exclusively with Docker.
In [December 2016][15], Kubernetes abstracted the container runtime implementation from the kubelet using an interface. This interface is called 'Container Runtime Interface', or 'CRI' for short.
![cri architecture](/images/2018-02-06-the-hitchhickers-guide-to-the-container-galaxy_3.png)

When the CRI was released, the first obvious implementation of this interface was naturally for Docker, this implementation is now called 'dockershim' (initially 'CRI-Docker') and is used by default with every default installation of Kubernetes.

After reading the chain of events in the Docker ecosystem, you might wonder how it affects the Kubernetes ecosystem. Well, every one of those container runtime variations found it's way into a CRI implementation. [rkt][16], [containerd][17], etc...

Another interesting CRI implementation if 'CRI-O' (CRI+OCI, originally called 'OCID'). This [Red Hat led project][17] tried to build a minimal CRI implementation, based on best-of-breed open source components, that is built for Kubernetes first.

[14]:https://cloudplatform.googleblog.com/2014/06/an-update-on-container-support-on-google-cloud-platform.html

[15]:http://blog.kubernetes.io/2016/12/container-runtime-interface-cri-in-kubernetes.html

[16]:https://kinvolk.io/blog/2017/11/announcing-the-initial-release-of-rktlet-the-rkt-cri-implementation/
[17]:http://blog.kubernetes.io/2017/11/containerd-container-runtime-options-kubernetes.html

[18]:https://www.redhat.com/en/blog/introducing-cri-o-10 

## Summary

The past few years have been super interesting for anyone who is into containers, but also super confusing. I hope that this post helped clarify some of the terms you might have heard and recalled "I know this has something to do with containers, but I'm not sure exactly how it relates to X/Y". One thing's for sure, the future is bright and intriguing.