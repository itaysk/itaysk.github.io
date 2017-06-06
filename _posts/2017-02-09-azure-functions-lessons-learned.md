---
title: Azure Functions - lessons learned
date: 2017-02-09 00:12
categories: [Technical-Howto]
tags: [azure-functions]
---

I recently did a small project which was based on Azure Functions. Although it was a small side project, it made me think a lot about Serverless architectures and Azure Functions, and I learned some valuable things about designing these kind of solutions. In this post I'll share the journey I have had, and lessons learned.

## Background
The project, [AzureGC](http://github.com/itaysk/azuregc) is a governance tool to help keep shared Azure subscriptions tidy. It's basically a bunch of automation scripts that track and automatically delete resources in Azure. The first version was built with Azure Automation, later I decided to migrate to Azure Functions.

## Phase 1 - naive port
First thing I did was to take the PowerShell code that was already running in Azure Automation, and just dump it into an Azure Function. Luckily Azure Functions support PowerShell as a first class citizen, so it basically just worked.

This naive port worked, but it was not very lean. It was a big chunk of code that was composed of 3 distinct steps that run synchronously, one after the other. For Azure Automation it was fine, but Functions, and Serverless in general promote a philosophy of very small functions that does one thing.  
This idea is also enforced to some level by the limited execution time (5 minutes at the moment) which the current port from Azure Automation already exceeded.

## Phase 2 - refactor functions
The next thing I did was to break that one large function into 3 smaller functions. again, this was not a big challenge as the code was already composed of 3 distinct parts. This is a best practice in software engineering anyway - refactoring pieces of code into functions. We do it all the time in "regular" codebases, same thing here. 

I did have to duplicate some common code like authentication and setup, but that wasn't a big deal.

## Phase 3 - more granular functions invocations
When all 3 code blocks lived together in the same function, they shared data by passing list collections from one stage to the other. After the break down into 3 functions, each function still had to get the data to work on in the form of list collections.  
An improvement to that was to make each function invocation work on a single data item, instead of a collection.  
This is also a best practice with Serverless design. When you refactor your code into smaller functions, try to also think about the data flow, and how to refactor it as well. A function that works on a list of items is fine by other standards, but in true serverless architecture we would want to refactor not only lines of code, but the items of data as well, and make each function invocation minimal. If you are familiar with Actor Model architectures, this idea will sound familiar. Once fully embraced, this design style will yield "Fan-Out" architecture, where the single point of entry is the function's trigger, and each layer produces tasks for the next layer.

So I introduced queues to communicate between the functions. A queue held tasks for execution by a dependent function. In example, after Phase 2 there was a function that decided which resources needed to be deleted, and another function that iterated on this collection and deleted resources one by one. After the current change, the job of the first function changed to just adding a messages to a queue for each resource to be deleted (1 message = 1 resource). Then, a deletion function was triggered by that queue messages and operated on a single item for each invocation.

This approach have two additional advantages:

- Scale out - when a function operated on a bunch of data, the platform (Azure Functions) couldn't scale out execution because that one function was the unit of deployment and execution. Now that each function handles 1 item, the platform can and will scale to as many function instances as needed to consume the queue in parallel.
- Resiliency -  A failure in a function invocation is isolated to the item it was processing.

## Phase 4 - use the platform's event binding system
Queues was in fact already in use in original code I ported from Azure automation. These queue operations was using the .NET client library for Azure Queues, which I loaded into PowerShell and used thanks to PowerShell's C# compatibility. Here's a sample of how it looked like:

```
[System.Reflection.Assembly]::LoadFrom(".\Microsoft.WindowsAzure.Storage.dll")
$queue = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueue -ArgumentList $QueueConnectionStringSasUrl
$queueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage -ArgumentList $message
$queue.AddMessage($queueMessage)
```

This worked and all but was not really using the full potential of Functions, which provide a native mechanism to use queues in the form of binding inputs and outputs.
So the next thing I did was to actually get rid of this entire code, and instead setup Function Output and Inputs for Azure Queue. Thankfully Azure Functions' native support for PowerShell makes the same code above as elegant as writing to a file:

```
$messages | ConvertTo-Json | Out-File -Encoding UTF8 $outputQueueItem
```
The lesson here is to rethink the way your functions interface with each other. It might not be queues, but files, or a DB, or anything else that your functions use for communication or share resources, even if indirectly.

## Phase 5 - independent functions
After heavy refactoring I was left with 3 functions that was small and lean, but still were inter-dependent.  
One function relied on the output of another function, or even worse on a side affect of another function. For example, one function treated resources in a certain way, and the next function relied on the fact that the first function had already run, and assumes items were already treated. The second function must run after the first function had finished, and relies on the fact that the first function succeeded.

I wanted to improve this design by making each function independent. There will still be a logical relation between functions in the form of communicating via the queue, but this would be a much looser coupling then before. This decoupling had 2 benefits:

- Functions scheduling was no longer synchronized. Each function can run on it's own time regardless of other functions.
- Functions now are idempotent, meaning it's not a big deal if the same function run twice, it would just have nothing to do.

## Phase 6 - event driven
The final step in this journey is something I haven't implemented yet due to technical constrains.  
Currently the entire system is based on a schedule. In each scheduled execution it needs to get the state of resource in Azure, so a function has to make an API call to Azure APIs (via Azure's PowerShell modules). 
What I would like to do is move to a more event driven architecture, where functions are activated by the Azure platform when new resources are created or changed.  
FaaS and Azure Functions are born for this, but unfortunately Azure Resource Manager (ARM) API, which is the data source for this system, did not fully support what needed for this to work.  
ARM does have an experimental webhook support, but it's going to be replaced soon so I'll just wait for the new version to implement this final step (you might be reading this post after this issue has been resolved so it's worth double checking that).

## Summary
I hope documenting my journey from a single script to a functional serverless application helped you better understand Serverless and Azure Functions, and learn the lessons that I have learned.  
To distill the core principles, I'd say that each Function should be:

- Small and single purposed (think of refactoring considerations in a regular codebase)
- Preferrably handles one piece of data instead of a large data set
- Utilize inputs, outputs, and triggers to their full potential
- Independent, does not rely on side affects of other functions, and preferably idempotent
- Event driven