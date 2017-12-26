---
title: The single use DeamonSet pattern and pre-pulling images in Kubernetes 
date: 2017-12-26 10:00
categories: [Technical-Howto]
tags: [kubernetes, docker]
canonical_url: https://codefresh.io/blog/single-use-daemonset-pattern-pre-pulling-images-kubernetes/ 
---

I was asked to find a way to to pre-pull Docker images into Kubernetes nodes. Meaning that when a pod is scheduled to run on a node, the relevant Docker image will already be on that node instead of being pulled at time of deployment. This can be useful in cases where the image is big, and pods are recreated often (for example in a scale out scenario).

## Docker in Docker
Creating a Docker container that does the `docker pull` command is pretty easy with 'Docker in docker' container (aka 'dind'):

```yaml
containers:
- name: prepull 
   image: docker
   command: ["docker", "pull", "hello-world"]
   volumeMounts:
     - name: docker
       mountPath: /var/run
volumes:
  - name: docker
     hostPath:
       path: /var/run
```
This pod will attach to the host's docker daemon and will pull the image 'hello-world' into the node it is running on.  
That's fine, but now we need to make sure this pod is being run on every node in the cluster.

## DaemonSet

That sounds like a good fit for a DaemonSet. This Kubernetes entity is suitable for infrastructure services that need to run on all nodes. The usual use cases are log collection and monitoring agents.  
It sounds prefect for our case, but the issue with DaemonSet, is that currently it can only have a `restartPolicy: Always`. Because our container does one task and then exit, it means that Kubernetes will restart the pod infinitely which will constantly attempt to pull the image (even if there's no change).

### DaemonSet single-use pattern

To work around this limitation, I did the following trick: I still used the same container to pull the image as explained before, but I designated it as an `initContainer` instead of regular `container`. This means it'll run only once when the pod is scheduled, and after it's done, the regular container in the pod will run. I then used the '[pause](https://groups.google.com/forum/#!topic/kubernetes-users/jVjv0QK4b_o)' container as the pod's main container. This container practically does nothing, which is exactly what we want to do after we pulled the image on that node - keep the DaemonSet pod alive but consume no resources.  
I guess you can call this pattern a workaround, but it's a useful trick for scenarios like this.

## Result

The end result is shared here: [https://gist.github.com/itaysk/7bc3e56d69c4d72a549286d98fd557dd](https://gist.github.com/itaysk/7bc3e56d69c4d72a549286d98fd557dd)

{% gist 7bc3e56d69c4d72a549286d98fd557dd %}

You can just `kubectl create` it, and it will make sure your 'hello-world' pod is pulled into every node in the cluster.
