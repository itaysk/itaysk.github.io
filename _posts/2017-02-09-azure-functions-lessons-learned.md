---
title: Azure Functions - lessons learned
date: 2017-02-09 00:12
categories: [Technical-Howto]
tags: [azure-functions]
---

I recently did a small project which was based on Azure Functions. Although it was a small side project, it made me think a lot about Serverless architectures and Azure Functions, and I learned some valuable things about designing these kind of solutions. In this post I'll try to share the journey I have had with this project, and lessons learned.

## Background
The project, [AzureGC](http://github.com/itaysk/azuregc) is a governance tool to help keep shared Azure subscriptions tidy. It's basically a bunch of fancy automation scripts that track and automatically delete resources in Azure. I have build the first version with Azure Automation, but then migrated to Azure Functions.

## Phase 1 - naive port
First thing I did was to take the PowerShell code that was already running in automation, and just port it into an Azure Function. Luckily Functions support PowerShell as a first class citizen, so I didn't have to make much changes.

This naive port worked, but it was not very lean. It was a big chunk of code that was composed of 3 main steps that run synchronously, one after the other. For Azure Automation it was fine, but Functions, and Serverless in general promote a philosophy of very small functions that does one thing.  
This idea is also enforced to some level by the limited execution time (5 minutes at the moment) which the current function script already exceeded.

## Phase 2 - refactor functions
So the next thing I did was to break the one large function into 3 smaller functions. again, this was not a big challenge as the code was already composed of 3 distinct parts. This is a best practice in software engineering anyway, refactoring pieces of code into functions. We do it all the time in "regular" codebases, same thing here. 

I did have to duplicate some code like authentication and setup, but that wasn't a big deal.

## Phase 3 - more granular functions invocations
When all 3 code blocks lived together in the same function, they shared data by passing list collections from one stage to the other. After the break down into individual functions, each function still had to get a bunch of data to work on. An improvement to that was to make each function invocation work on a single data item, instead of a data set. This is also a best practice with Serverless design.

I changed a bit how things worked by introducing a queues to communicate between the functions. A queue held tasks for execution by a dependent function. In example, after phase 2 there was a function that decided which resources needed to be deleted, and another function that iterated on this collection and deleted resources one by one. After this change the output of the first triage function was to add a messages in a queue for each resource to be deleted (1 message = 1 resource). Then, the deletion function was triggered by that queue and operated on a single item for each invocation.

## Phase 4 - use inputs and outputs
Queues help a role in the original code in Azure automation and now after phase 3 they were even more central to the solution. Up unitill now all queue operations was done using the .net client library for Azure Queues, which I loaded in PowerShell and used the C# api compatibility of PowerShell to operate. Here's a sample of how it looked like:

```
[System.Reflection.Assembly]::LoadFrom(".\Microsoft.WindowsAzure.Storage.dll")
$queue = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueue -ArgumentList $QueueConnectionStringSasUrl
$queueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage -ArgumentList $message
$queue.AddMessage($queueMessage)
```

This worked and all but was not really using the full potential of Functions - enter inputs and outputs.
So the next thing I did was to actually get rid of this entire code, and instead setup Function Output and Inputs as Azure Queues. Thankfully Azure Functions' native support for PowerShell makes the same code as easy as writing to a file:

```
$messages | ConvertTo-Json | Out-File -Encoding UTF8 $outputQueueItem
```

## Phase 5 - independent functions
After heavy refactoring I was left with 3 functions that was small and lean, but still were inter-dependent.  
One function relied on the output of another function, or even worse on a side affect of another function. For example, one function treated resources in a certain way, and the next function relied on the fact that the first function had already run continued that treatment. The second function had to run after the first function had finished, and relied on the fact that it did it's job with the shared data.

I wanted to fix this design by making each function independent. There is still a relation between functions in the way of communicating via queue, or sharing data, but this was a much looser coupling then before. This decoupling had 2 benefits:

1. Functions scheduling was no longer synchrinized. Each function can run on it's own time regardless of other functions.
2. Functions now are idempotemt, meaning it's not a bit deal if the same function run twice, it would just have nothing to do.

## Phase 6 - event driven
The final step in this journey is something I haven't implemented yet, but I know that I want to.  
Currently the functions that manipulate resources get this data by calling the Azure API (via the PowerShell modules). This is a pull model for getting data.  
What I would like to do is move to a more event driven architecture, where functions are activated by the Azure platform when new resources are created or changed. This is a push model for getting data.

Other then being more aligned with Serverless philosophy of event driven, and being even more agile, this has another important benefit - it makes sure function invocation is for handling a single data item and not a large data set. This is similar to what was done in Phase 3, but this time also for the data generating functions (as opposed to data consuming functions in Phase 3), making the entire system more agile.

The feasibility of this change depends on the characteristics of your data source. Not very data source can publish events for you to consume. In my case the data source is Azure Resource Manager API, which does have an experimental webhook support, but it's going to be replaced soon so I'll just wait for the new version to implement this final step.

## Summary
I hope documenting this journey from a single script to a composite serverless application helped you better understand Serverless and Azure Functions, and learn the lessons that I have learned.  
To distill the core principles, I'd say that each Function should be:

- Small and single purposed (think of refactoring considerations in a regular codebase)
- Preferrably handles one piece of data instead of a large data set
- Utilize inputs, outputs, and triggers to their full potential
- Independent, does not rely on side affects of other functions, and preferably idempotent
- Event driven

Not surprising I guess, but important to remember nonetheless.