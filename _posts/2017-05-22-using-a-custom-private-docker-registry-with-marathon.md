---
title: Using a private docker registry/repository with Marathon
date: 2017-05-22 19:47
categories: [Technical-Howto]
tags: [dcos, marathon, mesos, docker, azure-container-service, azure-container-registry]
---

I was using DC/OS-Marathon and needed to pull image from a private repository on Docker Hub or a custom registry like Azure Container Registry.  

## TL;DR
Follow this link: [https://mesosphere.github.io/marathon/docs/native-docker-private-registry.html](https://mesosphere.github.io/marathon/docs/native-docker-private-registry.html), but in step 2 replace the `uris` section with 

```
  "fetch" : [
    {
      "uri" : "https://path.to/file?including=querystring&parameters=andstuff",
      "extract" : true,
      "outputFile" : "dockerConfig.tar.gz" 
    }
  ]
```

---

## Deep Dive

The doc mentioned above is split in 2 steps. step 1 is fine: you `docker login` into your private repo or custom registry. This creates a config file with your credentials under `~/.docker/config.json`. You then package that up, and put it in a shared location. In my case I took the package and put it in Azure Blob Storage container. So far so good. 


In Step 2 of the doc, you should reference that shared package from your Marathon app configuration. Here I had some issues:

First, the doc instructs you to use `uris`, which are considered deprecated, and should be replaced with `fetch`. 

Second, I was planning to reference the file using Azure Storage [SAS uri](https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1). Which means the path to the file will include some query string parameters. That will bite us back in a moment.

If you followed the doc exactly, you archived the credentials file into `.tar.gz`. That's not absolutely necessary, but it's what the docs walks you through. If you did that, wou will have to add `extract:true` to the fetch section.

The [Mesos extraction process](http://mesos.apache.org/documentation/latest/fetcher/) checks the file extension and will not extract if the file doesn't have a known extension. Since I used a SAS uri, the file's extension was not picked up and Mesos failed to extract it.

A solution to this is to use Fetch's `outputFile` field, which will rename the downloaded file before attempting to extract.  
This is [not consistently documented](https://jira.mesosphere.com/browse/MARATHON-7298), but you could add `outputFile` to the fetch section and it will work fine.

The resulting marathon app definition looks like this:

```
{
  "id" : "...",
  "container" : {...},
  "fetch" : [
    {
      "uri" : "https://path.to/file?including=querystring&parameters=andstuff",
      "extract" : true,
      "outputFile" : "dockerConfig.tar.gz" 
    }
  ]
}
```