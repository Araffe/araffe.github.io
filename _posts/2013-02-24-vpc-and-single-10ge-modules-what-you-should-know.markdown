---
author: adraffe
comments: false
date: 2013-02-24 19:22:53+00:00
layout: post
link: http://adamraffe.com/2013/02/24/vpc-and-single-10ge-modules-what-you-should-know/
slug: vpc-and-single-10ge-modules-what-you-should-know
title: 'vPC and Single 10GE Modules: What You Should Know'
wordpress_id: 73
categories:
- Nexus 7000
- Virtual Port Channel
---

It's often a good recommendation to use multiple linecard modules in a switch chassis - this makes it possible to spread port-channels across linecards so if one fails, convergence times are kept to a minimum. If you are using vPC on the Nexus 7000, this recommendation becomes even more important. Why? Let's imagine we have a vPC setup with two Nexus 7000s and a downstream switch, connected via vPC. Each Nexus 7000 has a single M1 10GE linecard (used for both the peer-link and the upstream connections to the core), plus a single M1 1GE linecard (used to connect to the downstream switch, plus the Peer-Keepalive link):

[![vPC-single-10GE-mod-1](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-11.png?w=550)](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-11.png)

<!-- more -->Let's also assume N7K-1 is the primary vPC peer. Now let's see what happens if we lose one of the M1 10GE modules in N7K-1:

[![vPC-single-10GE-mod-2](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-2.png?w=550)](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-2.png)

What has happened here? Because the only available 10GE linecard in N7K-1 has failed, we have lost both the vPC Peer-Link and all our uplinks to the core from this switch. However, the M1 1GE card is still available, which means that both the vPC Peer-Keepalive and the vPC member port to the access switch are still up. Anyone familiar with vPC will at this point know that if the Peer-Link is down, but the Peer-Keepalive is still available, **the secondary vPC peer will disable its own vPC member links, **as follows:

[![vPC-single-10GE-mod-3](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-3.png?w=550)](http://adamraffe.files.wordpress.com/2013/02/vpc-single-10ge-mod-3.png)

Hopefully the issue with this is clear to see: the access switch has only one available path - via N7K-1. However N7K-1 has no available links to anywhere else in the network, therefore our access switch is now completely isolated.

How do we avoid this situation? There are two solutions:



	
  1. Purchase an additional 10GE module for each Nexus 7000

	
  2. Deploy vPC object tracking


Option 1 is ideal, however assuming that isn't feasible due to cost, we need to use vPC object tracking to get around this problem.

**What is vPC Object Tracking, and how does it help?**

Essentially, vPC Object Tracking is a method of signalling that a switch has lost a linecard and that the other vPC peer should assume the primary role (thereby keeping all its own vPC member links up). It does this using track lists, as follows:

_! Track the Peer-Link_
track 1 interface port-channel1 line-protocol_
! Track core uplinks_
track 2 interface Ethernet1/1 line-protocol
track 3 interface Ethernet1/2 line-protocol
!
track 10 list boolean or
object 1
object 2
object 3
!
vpc domain 1
track 10

In our example, if the above config is on N7K-1 and the 10GE module fails, N7K-1 effectively says "Hey, N7K-2, I've lost my Peer-Link and core uplinks. This means I must have lost my only 10GE linecard - I need you to take over the vPC primary role". N7K-2 then assumes the primary role, which means that it will not bring down its own vPC member links. This means that traffic continues to flow from the access switch to second Nexus 7000 - problem solved!
