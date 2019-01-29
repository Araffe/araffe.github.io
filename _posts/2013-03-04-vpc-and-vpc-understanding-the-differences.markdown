---
author: adraffe
comments: false
date: 2013-03-04 16:58:40+00:00
layout: post
link: http://adamraffe.com/2013/03/04/vpc-and-vpc-understanding-the-differences/
slug: vpc-and-vpc-understanding-the-differences
title: 'vPC and vPC+: Understanding the Differences'
wordpress_id: 256
categories:
- Virtual Port Channel
tags:
- FabricPath
- Nexus
- vPC
---

Virtual Port Channel (vPC) is a technology that has been around for a few years on the Nexus range of platforms. With the introduction of FabricPath, an enhanced version of vPC, known as vPC+ was released. At first glance, the two technologies look very similar, however there are a couple of differences between them which allows vPC+ to operate in a FabricPath environment. So for those of us deploying FabricPath, why can't we just use regular vPC? <!-- more --> Let's look at an example. The following drawing shows a simple FabricPath topology with three switches, two of which are configured in a (standard) vPC pair. [![vPC-vPC+]({{ site.baseurl }}/img/2013/03/vpc-vpc3.png?w=550)]({{ site.baseurl }}/img/2013/03/vpc-vpc3.png) A single server (MAC A) is connected using vPC to S10 and S20, so as a result traffic sourced from MAC A can potentially take either link in the vPC towards S10 or S20. If we now look at S30's MAC address table, which switch is MAC A accessible behind? The MAC table only allows for a one to one mapping between MAC address and switch ID, so which one is chosen? Is it S10 or S20? The answer is that it could be _either_, and it is even possible that MAC A could 'flip flop' between the two switch IDs. So, clearly we have an issue with using regular vPC to dual attach hosts or switches to a FabricPath domain. How do we resolve this? We use vPC+ instead. vPC+ solves the issue above by introducing an additional element - the 'virtual switch'. The virtual switch sits 'behind' the vPC+ peers and is essentially used to represent the vPC+ domain to the rest of the FabricPath environment. The virtual switch has its own FabricPath switch ID and looks, for all intents and purposes, like a normal FabricPath edge device to the rest of the infrastructure. [![vPC-vPC+2]({{ site.baseurl }}/img/2013/03/vpc-vpc2.png?w=550)]({{ site.baseurl }}/img/2013/03/vpc-vpc2.png) In the above example, vPC+ is now running between S10 and S20, and a virtual switch - S100 - now exists behind the physical switches. When MAC A sends traffic through the FabricPath domain, the encapsulated FP frames will have a source switch ID of the virtual switch, S100. From S30's (and other remote switches) point of view, MAC A is now accessible behind a single switch - S100. This enables multi-pathing in both directions between the Classical Ethernet and FabricPath domains. Note that the virtual switch needs a FabricPath switch ID assigned to it (just like a physical switch does), so you need to take this into account when you are planning your switch ID allocations throughout the network. For example, each access 'Pod' would now contain three switch IDs rather than two - in a large environment this could make a difference. Much of the terminology is common to both vPC and vPC+, such as Peer-Link, Peer-Keepalive, etc and is also configured in a very similar way. The major differences are:



	
  * In vPC+, the Peer-Link is now configured as a FabricPath core port (i.e. _switchport mode fabricpath_)

	
  * A FabricPath switch ID is configured under the vPC+ domain configuration (_fabricpath switch id _) - remember to configure the same Switch ID on both peers!

	
  * Both the vPC+ Peer-Link and member ports must reside on F series linecards.


vPC+ also provides the same active / active HSRP forwarding functionality found in regular vPC - this means that (depending on where your default gateway functionality resides) either peer can be used to forward traffic into your L3 domain. If your L3 gateway functionality resides at the FabricSpine layer, vPC+ can also be used there to provide the same active / active functionality.
