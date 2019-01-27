---
author: adraffe
comments: true
date: 2017-11-22 11:19:52+00:00
layout: post
link: http://adamraffe.com/2017/11/22/the-wonderful-world-of-azure-cli-jmespath-queries/
slug: the-wonderful-world-of-azure-cli-jmespath-queries
title: Azure CLI JMESPath Queries
wordpress_id: 1674
categories:
- Azure
---

I use the Azure CLI for much of what I do in Azure now - true, the same things can usually be achieved through the portal or by using PowerShell, but I just prefer the Linux / Bash nature of the CLI.

One of the things that makes the CLI so nice to use is the powerful query language that it has available - this language is called **JMESPath**. JMESPath isn't specific to the Azure CLI though - it's a query language for JSON ([http://jmespath.org/](http://jmespath.org/)) so it can be used whenever you need to manipulate or query JSON data.<!-- more -->

So how can we use it to improve our Azure CLI operations? Let's start with a simple example. I've decided that I want to use the CLI to find out the names of all the virtual machines I have running in a particular resource group. I'm going to start by running the command **az vm list -g demo.VMs** to return a list of all VMs in the 'demo.VMs' resource group (by the way, I'm using the [Azure Cloud Shell](https://azure.microsoft.com/en-gb/features/cloud-shell/) to run these commands - check it out, it's great):

![Queries-1](https://adamraffe.files.wordpress.com/2017/11/queries-1.jpg)

Woah! I just got a ton of JSON back from that command. What you see in this screenshot is just the very top of the output for one VM only - this goes on for many more pages. But all I wanted was to find out the names of my VMs! So how do I narrow this down to the information I want? Here's where JMESPath queries come in.

I'll add a query to the original command that should give me only the names of the VMs. The command I'll run is as follows:

    
     <strong>az vm list -g demo.VMs --query [].[name]</strong>


This gives me back something much more civilised:

![Queries-2](https://adamraffe.files.wordpress.com/2017/11/queries-2.jpg)

Not bad, but still a bit messier than I would like - let's try this again with the **-o table** switch at the end of the command:

![Queries-3](https://adamraffe.files.wordpress.com/2017/11/queries-3.jpg)

OK, that looks better.

Now, I've decided that - along with the VM name - I want to know the name of the operating system disk attached to each machine. I need to add it to the query, but how do I know what to add? Let's take a look at part of the original JSON query from one of the VMs:

![Queries-4](https://adamraffe.files.wordpress.com/2017/11/queries-41.jpg)

From the above, it looks like the field I am looking for (name) is buried under the 'storageProfile' object and then under 'osDisk'. So let's add this to the query and see what happens:

    
    <strong>az vm list -g demo.VMs --query [].[name,storageProfile.osDisk.name] -o table</strong>


![Queries-5](https://adamraffe.files.wordpress.com/2017/11/queries-5.jpg)

Nice! I can now see the name of the VMs and the OS disk used by each one. However, I'm still not happy that the column headings in my tables are simply labelled 'Column1', 'Column2', etc. To add a nice friendly column heading, I can add the heading I want to the query as follows:

    
    <strong>az vm list -g demo.VMs --query "[].{Name:name,Disk:storageProfile.osDisk.name}" -o table</strong>


![Queries-6](https://adamraffe.files.wordpress.com/2017/11/queries-6.jpg)

Perfect. Note that this time, I have used curly brackets for the second part of the query, plus I have enclosed the whole query in quotation marks.

In my examples so far, I'm getting information back about both the Linux and Windows VMs that live in my resource group. The problem is, I'm only really interested in the Windows VMs - so how do I narrow this query down even further to only include the Windows machine? Well, we can set up the query to look only for the elements in our array containing a certain value - in this case, we want to make sure that only the elements that contain 'Windows' make it into our output. Here's how it's done:

    
    <strong>az vm list -g demo.VMs --query "[?contains(name, 'Windows')].{Name:name,Disk:storageProfile.osDisk.name}" -o table</strong>


This gives us the following:

![Queries-7](https://adamraffe.files.wordpress.com/2017/11/queries-7.jpg)

Now let's take this a bit further. Suppose I want to get a list of all the VMs _not _currently running (i.e. deallocated) and with 'Linux' in the name - and then start those VMs. One way of achieving this is to do the following:

    
    <strong>az vm list -g demo.VMs --show-details --query "[?contains(name, 'Linux') && powerState == 'VM deallocated']".id -o tsv | xargs -L1 bash -c 'az vm start --ids $0'</strong>


There's a bit going on here, so let's break it down. In the first part of the command, we run the **az vm list **command, but this time we add the **--show-details** parameter (only this extended version of the command shows the power state of the virtual machine). Then we add a query that returns only those VMs that a) have 'Linux' in the name and b) have a current power state of 'VM deallocated'. We also want to make sure that we return only the ID of the VM - hence the **.id** on the end of the query. Now the table output format that we've been using up until now isn't going to work here, so we'll need to use a different output format - in this case we're going to use the tab separated output format (**-o tsv**) instead.

In the second part of the command, we're taking the output of the first command (which returns the IDs of the VMs we are interested in) and piping this to the **az vm start** command. The **xargs** command is used to pass the output values from the first command as input values to the second.

There's a whole lot more you could do with JMESPath queries - check out the JMESPath site [here](http://jmespath.org/) for more info.

Also, my colleague Rich Cheney put together a self paced lab guide around Azure CLI, BASH and JMESPath - check it out [here](https://azurecitadel.github.io/guides/cli/).

Thanks for reading!


