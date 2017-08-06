---
title: Why I love PowerShell
date: 2017-08-05 13:42
categories: [Technical-Howto]
tags: [powershell, bash]
---

> This demo script was a small part of a session I gave about dev tools and productivity tips. The session received positive feedback from a folks who were distant from the Microsoft realm and were quite surprised to experience PowerShell, so I decided to post it also here. This post is a basic level introductory, and not even that because I just highlight a feature I like.

One of the nice things about PS is that everything is an Object (like other OO languages like .NET or Java). This means that every command argument is an object, and more importantly every return value is an object.  
This design makes manipulating command results and piping between commands a breeze compared to other shell environments (including bash) in which everything is text, including command arguments, and most annoyingly - command return values.  
With bash, if a command returned a table of values, you’ll have to visually parse through that table to extract meaningful data. In contrast, with PowerShell you might see a table printed on the screen, but you actually get is a structured collection that you could index into, iterate over, and more.

Let’s run a simple cmdlet - `Get-Item *`. This is the `ls` of PS world. We get all the file system entries in the current location (`/`).

![](/images/2017-08-05-why-i-love-powershell_1.png)

We see a nice table presented on the screen.  
Let’s capture this array: `$items = Get-Item *` and make sure by printing it: `$items`.

![](/images/2017-08-05-why-i-love-powershell_2.png)

Now we can examine it’s type: `$items.GetType()`. 

![](/images/2017-08-05-why-i-love-powershell_3.png)

It's an `Array`! So we can index into an element: `$items[3]`, and also examine this one’s type: `items[3].GetType()`. 

![](/images/2017-08-05-why-i-love-powershell_4.png)

So that list we saw on screen was actually an `Array` of `DirectoryInfo` objects.

In fact, we can `GetType()` on every object we come across because it’s an Object Oriented shell and everything is an Object, and all objects inherit from a base Object which defines the `GetType()` method.

The real power of the object oriented nature of PS reveals when we try to manipulate objects. Let’s try:  
`$items | where LastWriteTime -ge ’07-01-2017’`

![](/images/2017-08-05-why-i-love-powershell_5.png)

This was too easy! What happened here:  
We piped the array of DirectoryInfo objects into the `Where-Object` cmdlet (alias `where`), and defined a predicate of LastWriteTime greater-then a date. The date implicitly cast from string to a DataTime object. We could have passed a DateTime object as well.

BTW - notice how auto-completion worked while looking for the members of the DirectoryInfo object. Remember, this is typed objects and classes, meaning the tooling is awesome. We can use tab all around without autocompletion scripts (like in bash) and even do ctrl+space for intellisense!  
Let’s look at the LastWriteTime field: `$items[3].LastWriteTime.` and press Tab - you’ll see all of the DateTime object members.

![](/images/2017-08-05-why-i-love-powershell_6.png)

To complete the demo lets pretty print the results by: `$items | where LastWriteTime -ge ’07-01-2017’ | select Name`. 

![](/images/2017-08-05-why-i-love-powershell_7.png)

Good luck GREPing your way through that! This distilled output could be piped into another cmdlet as an input. Now we are starting to realize the power of PS.

That was a simple demo of playing with full system entries, but remember that everything in PS is an object so the same tools will serve you for every interaction you have with the shell. `Get-Process` for example is exactly the same.

![](/images/2017-08-05-why-i-love-powershell_8.png)

One last thing I’ll mention, is that PS is based on .NET, which means you can load .NET assemblies, and use the entire .NET framework and ecosystem (nuget) within PS, Which is huge…

### No one is perfect

There are some things I dislike with PowerShell, most notable, that cmdlets are flat instead of hierarchical. For example ‘Get-Item’, ‘New-Item’, ‘Move-Item’, etc. are independent and not grouped in some parent ‘Item’ construct.   
This design manifests itself when the hierarchy is deeper, like in Azure: `Get-AzureRmResourceGroupDeploymentOperation` - this is a single command that’s actually 4 levels deep a hierarchy: AzureRm->ResourceGroup->Deployment->Operation.   
If I wanted to find out what commands a deployment has, or to navigate through the resource group level, it would have been much harder. This is not the end of the world, but it makes exploring cmdlets/modules a pain.  
Unix world (bash inherently) has a hierarchical convention where commands are friendlier (IMO) for exploring. The equivalent of the above PS will be: `az group deployment operation show`. If I’m at ‘deployment’ and not sure what follows, I can just hit enter or `—help` and see what’s in this level.

### PS is cross platform and Open Source

The sharp eyed among you might have noticed that I'm running PowerShell in Docker, on a Mac. Yeap - that's true. PS is available for Windows, Linux, and Mac, and is available on GitHub, here: [https://github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)