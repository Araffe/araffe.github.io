---
author: Adam
comments: true
date: 2017-09-28 15:33:49+00:00
layout: post
link: http://adamraffe.com/2017/09/28/secure-connectivity-from-azure-vnets-to-public-services/
slug: secure-connectivity-from-azure-vnets-to-public-services
title: Secure Connectivity from Azure VNets to Public Services
wordpress_id: 1526
categories:
- Azure
- Security
---

A question I've heard a few times recently is "_if I have services running in an Azure Virtual Network, how do I securely connect that VNet to Azure public services, such as Blob Storage?_". Microsoft have this week announced a couple of features designed to help with this scenario, but before delving into those, let's look at the issue we are actually trying to solve.

First, a few basics: a Virtual Network (VNet) is a private, isolated, network within Azure into which you can deploy infrastructure resources such as virtual machines, load balancers and so on:

![Secure-VNets1](https://adamraffe.files.wordpress.com/2017/09/secure-vnets11.jpg)

Although these VMs can (and very often do) have direct Internet access, it is of course possible to restrict connectivity into and out of this VNet according to your requirements.<!-- more -->

Now consider the myriad of public services available in Azure, such as Blob Storage and Azure SQL. As these services are public, they do _not _sit inside a virtual network and are not isolated from the "outside world". The question is, if I want to connect from a VM inside a VNet to a public service such as Blob Storage, how does that work? It's actually pretty straightforward - VMs connect to Blob Storage or other Azure services using the normal Internet facing end points (e.g. _<storageaccount>.blob.core.windows.net_), as shown in the following diagram:
![Secure-VNets2.jpg](https://adamraffe.files.wordpress.com/2017/09/secure-vnets22.jpg)

OK, that's fine - but doing it this way does open up a couple of issues:



	
  * The VM needs Internet access - what if I want to prevent the VM from having Internet access but retain access to the Azure services only?

	
  * The storage account in question is open to anyone else on the public Internet (true, there are various authentication mechanisms such as SAS tokens, but the storage account URL is still fundamentally 'open' and not as private as we would like).


Let's deal with these one at a time.

**How do I structure Network Security Groups to allow access only to Azure services?**

In this scenario, I want my VM to have access to Azure Blob Storage, however I _don't_ want that VM to be able to access the wider Internet. Network Security Groups (NSGs) seem like the obvious choice here, but what addresses should I allow or deny in the rules? Do I allow access to the specific [Azure data centre IP ranges](https://www.microsoft.com/en-gb/download/details.aspx?id=41653) for storage and deny everything else? What happens if those IPs change at any point?

Luckily, there is now a solution to this issue: [NSG Service Tags](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview#service-tags), which were announced this week and are currently in preview. The idea behind Service Tags is simple: when defining NSG rules, you use the Service Tag for the service you are interested in as the destination, instead of IP addresses. So in my example above, I would have an NSG rule allowing access from my VNet to the storage service in my region (defined using a Service Tag), followed by another rule denying all other Internet access from that VNet.

The rule allowing access to storage would look like this:

![Secure-VNets3](https://adamraffe.files.wordpress.com/2017/09/secure-vnets31.jpg)

So by using NSG Service Tags, we can simply specify the Azure public service we want to give access to without having to worry about IP addresses - much easier.

**Public services have Internet reachable IP addresses and are therefore not truly 'private'.**

Our second issue is that - regardless of the authentication mechanisms we implement - the IP addresses for our Azure public services are fundamentally reachable by the entire Internet and therefore cannot be considered truly 'private'. Wouldn't it be much nicer if we could have a private connection between our Virtual Network and the service in question? Well, we're in luck - also announced this week were [VNet Service Endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview), a method of extending private address space to Azure services, such as storage and SQL.

To enable this, the first thing we do is enable Service Endpoints on the VNet (in this case, I'm using storage as an example):

![Secure-VNets4](https://adamraffe.files.wordpress.com/2017/09/secure-vnets4.jpg)

The second thing we need to do is to turn off access from any network on the storage account and only allow the specific VNets in question. This is done from the storage account configuration:

![Secure-VNets5](https://adamraffe.files.wordpress.com/2017/09/secure-vnets5.jpg)

Now that that's configured, the VM within my VNet / subnet can access objects within the storage account, while that same account can no longer be accessed from the Internet:

![Secure-VNets6](https://adamraffe.files.wordpress.com/2017/09/secure-vnets6.jpg)

If we take a look at the effective routes associated with my virtual machine's NIC, we can see that we have some additional routes added for the service endpoint:

![Secure-VNets7](https://adamraffe.files.wordpress.com/2017/09/secure-vnets7.jpg)

One other interesting point is that I can no longer access the storage account even from within the Azure portal:

![Secure-VNets8](https://adamraffe.files.wordpress.com/2017/09/secure-vnets8.jpg)

That's it - hopefully it's clear from this post how these new features make securing access to Azure public services much easier. Thanks for reading!




