---
title: Create SQL DB from ARM template - requestedServiceObjectiveId
date: 2015-09-04 23:56
categories: [Technical-Howto]
tags: [ARM, Azure, SQL]
---

If you try to create a Azure SQL Database from a ARM template, you probably went through [the docs](https://msdn.microsoft.com/en-us/library/azure/mt163685.aspx),  encountered in the mandatory property called `requestedServiceObjectiveId` which is defined as "The performance level for the database". That's essentially the [pricing tier](http://azure.microsoft.com/en-gb/pricing/details/sql-database/) that you choose for your DB. Now that we understand that, we can set a value for the field, but then we find out that the values for this property are GUIDs that correlate to the available performance levels. [the docs](https://msdn.microsoft.com/en-us/library/azure/mt163685.aspx) shows this example:
`"requestedServiceObjectiveId":"26e021db-f1f9-4c98-84c6-92af8ef433d7"`
Those GUIDs are currently very hard to find. The upcoming Azure PowerShell 0.9.8 [will include](https://github.com/Azure/azure-powershell/pull/792) a cmdlet for that. But even so, why using GUIDs when we can use names...

## A better solution
There's an undocumented property called `requestedServiceObjectiveName` which allows you to use the pricing tier name like:
`"requestedServiceObjectiveName": "S1"`
As I said it's not documented but confirmed working and makes your template a lot more easy to both write and read.
