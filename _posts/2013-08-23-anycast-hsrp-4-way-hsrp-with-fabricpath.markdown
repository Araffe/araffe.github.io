---
author: Adam
comments: false
date: 2013-08-23 07:19:06+00:00
layout: post
link: http://adamraffe.com/2013/08/23/anycast-hsrp-4-way-hsrp-with-fabricpath/
slug: anycast-hsrp-4-way-hsrp-with-fabricpath
title: 'Anycast HSRP: 4-way HSRP with FabricPath'
wordpress_id: 627
categories:
- Nexus 7000
tags:
- FabricPath
- Nexus
---

One of the interesting design considerations when deploying FabricPath is where to put your layer 3 gateways (i.e. your SVIs). Some people opt for the spine layer (especially in smaller networks), some may choose to deploy a dedicated pair of leaf switches acting as gateways for all VLANs, or distribute the gateway function across multiple leaves. Whatever choice is made, there are a couple of challenges that some have come across.

As you may know, vPC+ modifies the behaviour of HSRP to allow either vPC+ peer to route traffic locally. This is certainly useful functionality in a FabricPath environment as it allows dual active gateways, however what if you want your default gateways on the Spine layer (where there are no directly connected STP devices, and therefore no real need to run vPC+)? What you end up with in this case is vPC+ running on your Spine switches in order to gain access to the dual active HSRP forwarding, _but with no actual vPC ports on the switch_. This works fine - but most people would prefer not to have vPC+ running on their Spine switches if they can avoid doing so.

[![Anycast-HSRP-1]({{ site.baseurl }}/img/2013/08/anycast-hsrp-1.png)]({{ site.baseurl }}/img/2013/08/anycast-hsrp-1.png)<!-- more -->

The other drawback of using vPC+ is that you can only have two active gateways - only HSRP peers in 'active' or 'standby' state will forward traffic, with peers in 'listening' state unable to forward. So in larger FabricPath networks where you have multiple Spine switches, lots of bandwidth, many different layer 2 paths, etc, you are still limited to two active L3 gateways.

As of the NX-OS 6.2(2) release, there is a new option - Anycast HSRP. Essentially, this feature allows you to run multiple active first-hop gateways without the use of vPC+, as well as provide a higher number of active forwarders (currently 4). So how does it work?

When you configure vPC+, you use an Emulated Switch ID to represent both peers to the network - this switch ID is configured on both peers and other FP switches on the network use that ESID to reach MAC addresses (including the HSRP VMAC) behind the two switches. With Anycast HSRP, a similar concept is used, known as the _Anycast Switch ID (ASID)_. This is for all intents and purposes identical to the vPC+ Emulated Switch ID, except that a) you don't need to configure vPC+ and b) it works across more than two switches. To configure an ASID, we use a new type of configuration on the switch - the HSRP bundle. The configuration looks like this:

    
    hsrp anycast 1 ipv4
      switch-id 102
      vlan 10,11,12
      priority 120
      no shutdown


Once you have added the bundle configuration, it then needs be tied to an interface:

    
    interface Vlan10
       hsrp version 2
       hsrp 1
       ip 1.2.3.4


Note that HSRP version 2 must be configured on the interface for Anycast HSRP to work.

Let's say we now have four spine switches in our network and want to run HSRP across all four of them. The resulting topology looks like this:

[![Anycast-HSRP-2]({{ site.baseurl }}/img/2013/08/anycast-hsrp-2.png?w=550)]({{ site.baseurl }}/img/2013/08/anycast-hsrp-2.png)

In the above scenario, the two leaf switches (S10 and S20) learn that the HSRP VMAC is accessible via the switch ID S102 (the Anycast switch ID). FabricPath IS-IS then calculates that this switch ID is accessible via any of the spine switches (S1, S2, S3 or S4). Note also that all four spine nodes (even the ones in HSRP listening mode) are actively forwarding L3 traffic.

Note that there is one important consideration when using Anycast HSRP - **all switches must be running Anycast HSRP capable software**. So in our example above, we must also be running an Anycast HSRP capable image on the leaf switches (S10 and S20) as well as the spines. In many networks, the leaf switches will be Nexus 5500 / Nexus 6000 switches; support for Anycast HSRP will be coming to those platforms in a future release.

