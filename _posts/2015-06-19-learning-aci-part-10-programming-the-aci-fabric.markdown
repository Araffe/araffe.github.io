---
author: Adam
comments: true
date: 2015-06-19 16:49:09+00:00
layout: post
link: http://adamraffe.com/2015/06/19/learning-aci-part-10-programming-the-aci-fabric/
slug: learning-aci-part-10-programming-the-aci-fabric
title: 'Learning ACI - Part 10: Programming the ACI Fabric'
wordpress_id: 990
categories:
- ACI
- Nexus 9000
- SDN
tags:
- APIC
- sdn
---

Everything I've shown so far in this blog series has been focused on using the APIC GUI to achieve what we need. This is fine for small environments or for becoming familiar with ACI, but what if you need to configure 100 tenants, each with 50 EPGs, tens of private networks and bridge domains, multiple contracts to allow traffic and a L3 Outside or two thrown in? Configuring all of that through the GUI is clearly going to be massively time consuming and probably quite error prone, so we need to find a better way of doing things. ACI has a wide variety of programmability options that can be used to automate the provisioning of the fabric and its policies.

One particularly strong aspect of ACI is how easy it is for network engineers to start interacting with the fabric in a programmatic manner. Speaking from personal experience, it's incredibly easy for someone with a 'traditional' network engineering background with limited development experience to start using the programmability features of ACI. With that said, let's take a look at some of the options we have around ACI programmability:


  * Native REST API using XML or JSON

	
  * ACI Python SDK ("Cobra")

	
  * ACI Toolkit


Let's take a look at the native REST API option first.

**Native REST API**

The first option - and arguably the one most people look at first - is to interact with the APIC API using raw XML or JSON. Using raw XML / JSON has the advantage of not requiring the user to have any real programming knowledge (for example, Python) and interacting with the REST API natively can be done through a variety of tools (e.g. any tool capable of posting data to a URL such as cURL, Postman plugin for Chrome, etc). So how do we get started with this?

Before you start, it's worth bearing in mind that to use the REST API effectively, it's useful to have some understanding of how the object model works. For example, if we want to create a tenant, private network, bridge domain and EPG using the API, we need to have a bit of knowledge about how these objects relate to each other. Fortunately, the APIC provides some tools to help us understand this - one of which is the object store browser (Visore).

To get to Visore, browse to <APIC-IP-address>/visore.html and log in. Once in, you'll be able to query for classes or objects using the search box at the top of the screen. In this example, I'm going to search for all instances of class _fvTenant:_

[![Visore-fvTenant]({{ site.baseurl }}/img/2015/06/visore-fvtenant.png)]({{ site.baseurl }}/img/2015/06/visore-fvtenant.png)

In the above screenshot, my query has returned 32 objects of class fvTenant (I have 32 tenants on my fabric), two of which I have shown here. From here, you can see the distinguished name of each tenant object (e.g. _uni/tn-LAN01_). Now, the really useful part of this is that clicking the right arrow (>) next to the distinguished name will take you to another screen showing all of the child objects of the tenant object in question:

[![Visore-children]({{ site.baseurl }}/img/2015/06/visore-children.png)]({{ site.baseurl }}/img/2015/06/visore-children.png)

In the above screenshot, you can see two of the child objects for my tenant (named "adraffe-test") - specifically, we see an application profile (class fvAp, named "Test-App-1") and a bridge domain (class fvBD, named "Adam-Test-BD"). Scrolling further down reveals additional objects such as private networks, contracts and filters. Clicking the right arrow next to the DN takes you further down the tree - for example, drilling further into the "fvAp" object here would take me to the EPG objects contained within.

Now that we have some knowledge of the object model, we can start with some basic interaction with the REST API. The first thing I need to do is authenticate, which I can do by posting to the URL in the screen shot below and sending the XML shown:

[![aaaLogin]({{ site.baseurl }}/img/2015/06/aaalogin.png)]({{ site.baseurl }}/img/2015/06/aaalogin.png)

Now that I've authenticated, I can start creating some objects through the ACI. I'll start by creating a new tenant named 'Finance':

[![Tenant-Create]({{ site.baseurl }}/img/2015/06/tenant-create.png)]({{ site.baseurl }}/img/2015/06/tenant-create.png)

That's it - sending that simple piece of XML to the APIC created a new tenant. Now let's create a private network, bridge domain, application profile and EPG inside my new tenant. I post to the same URL as in the last example (the "uni" in the URL is used to represent "policy universe"), but this time I send the following:

[![Tenant XML]({{ site.baseurl }}/img/2015/06/tenant-xml.png)]({{ site.baseurl }}/img/2015/06/tenant-xml.png)

Let's take a closer look at what the above XML does. Firstly, I need to include the parent tenant object I want to modify (Finance). I then create a new private network (fvCtx) named _Finance-Net_. Next, I create a bridge domain (fvBD) named _Finance-BD_, with a subnet address (fvSubnet) of 10.1.1.1/24. I also create a relationship between my new bridge domain and the private network I created earlier (using fvRsCtx). Next up, I create an application profile (fvAp) called _Finance-AP1_ and add a new EPG (fvAEPg) to it named _Finance-EPG_. Finally, I create a relationship from my new EPG to the BD I created earlier (using fvRsBd).

This is extremely easy to get the hang of - you can use the API inspector and "Save As" tools which I referred to in my [earlier post](http://adamraffe.com/2014/12/03/learning-aci-part-3-getting-familiar-with-the-apic/) (as well as Visore) to help you to get familiar with the object model and build your own scripts.

**ACI Python SDK (Cobra)**

If you are familiar with Python, Cisco have a Python SDK (also known as "Cobra") available. This SDK allows you to build a Python program to make API calls to the APIC without having to post raw XML or JSON as in the last example. You still need a working knowledge of the object model, which again the tools mentioned above can help with.

As a comparison, let's take a look at the equivalent Python code which would create the same tenant, private network, bridge domain and EPG we created earlier (note that there is more to this script not shown here, such as importing the relevant modules, logging into the APIC, etc):

[![Cobra-Code]({{ site.baseurl }}/img/2015/06/cobra-code.png)]({{ site.baseurl }}/img/2015/06/cobra-code.png)

Anyone not familiar with Python may at this point be wondering if they should steer clear of the Python SDK - however, fear not: there is an extremely cool tool which will take the XML or JSON which you provide and automatically generate Python code for you. This tool is called _arya_ (APIC REST Python Adapter) and is available here:

https://github.com/datacenter/arya

You can see this tool in action by simply downloading some XML / JSON from the APIC using the "Save As" feature and then feeding the file into the arya tool. The tool will generate the equivalent Python code for the XML or JSON you provided. Pretty cool, right?

**ACI Toolkit**

The final option I'll look at here is the ACI Toolkit. One of the considerations for using either the native REST API or the Cobra SDK is that you do need to have some familiarity with the ACI object model. That's not necessarily a huge mountain to climb, but some people may be looking for a quicker and simpler way of accessing the programmability features available with ACI. Step forward, ACI Toolkit.

The ACI Toolkit is essentially a set of Python libraries which takes the ACI object model and abstracts it into a greatly simplified version. Right now, the ACI Toolkit doesn't provide full feature coverage in the same way that the native API or Cobra SDK does, but if you are looking for a simple way to create the most commonly used objects (EPGs, BDs, private networks and so on) then it's worth taking a look at what the Toolkit can offer.

Here is an example of how the ACI Toolkit can be used to create some common objects within ACI:

[![Toolkit]({{ site.baseurl }}/img/2015/06/toolkit.png)]({{ site.baseurl }}/img/2015/06/toolkit.png)

One other nice bonus of the ACI Toolkit is that a number of applications are thrown in, such as the Snapback app (used for config rollback within ACI), End Point Tracker (used to keep track of and display end point information within the fabric), various visualisation tools and more.

You can get more info on the ACI Toolkit, including instructions to install [here](http://datacenter.github.io/acitoolkit/docsbuild/html/).

To sum up, hopefully it's clear from this post that programmability in ACI is not limited to developers - anyone with a background in traditional networking should be able to get up to speed reasonably quickly and be able to start taking advantage of the benefits associated with programmability.

Thanks for reading.
