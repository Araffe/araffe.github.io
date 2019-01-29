---
author: adraffe
comments: false
date: 2013-03-12 20:28:08+00:00
layout: post
link: http://adamraffe.com/2013/03/12/fabricpath-for-layer-2-dc-interconnect/
slug: fabricpath-for-layer-2-dc-interconnect
title: FabricPath for Layer 2 DC Interconnect?
wordpress_id: 373
tags:
- FabricPath
- Nexus
---

The requirement for layer 2 interconnect between data centre sites is very common these days. The pros and cons of doing L2 DCI have been discussed many times in other blogs / forums so I won't revisit that here, however there are a number of technology options for achieving this, including EoMPLS, VPLS, back-to-back vPC and OTV. All of these technologies have their advantages and disadvantages, so the decision often comes down to factors such as scalability, skillset and platform choice.

Now that FabricPath is becoming more widely deployed, it is also starting to be considered by some as a potential L2 DCI technology. In theory, this looks like a good bet - easy configuration, no Spanning-Tree extended between sites, should be a no brainer, right? Of course, things are never that simple - let's look at some things you need to consider if looking at FabricPath as a DCI solution.<!-- more -->

**1: FabricPath requires direct point-to-point WAN links. **

A technology such as OTV uses mac-in-IP tunnelling to transport layer 2 frames between sites, so you simply need to ensure that end-to-end IP connectivity is available. As a result, OTV is very flexible and can run over practically any network as long as it is IP enabled. FabricPath on the other hand requires a direct layer 1 link between the sites (e.g. dark fibre), so it is somewhat less flexible. Bear in mind that you also lose some of the features associated with an IP network - for example, there is currently no support for BFD over FabricPath.

**2: Your multi-destination traffic will be 'hairpinned' between sites.
**

In order to forward broadcast, unknown unicast and multicast traffic through a FabricPath network, a multi-destination tree is built. This tree generally needs to 'touch' each and every FP node so that multi-destination traffic is correctly forwarded. Each multi-destination tree in a FabricPath network must elect a root switch (this is controllable through root priorities, and it's good practice to use this), and all multi-destination traffic must flow through this root. How does this affect things in a DCI environment? The main thing to remember is that there will generally be a single multi-destination tree spanning both sites, and that the root for that tree will exist _on one site or the other_. The following diagram shows an example.

[![FP-DCI-1]({{ site.baseurl }}/img/2013/03/fp-dci-11.png?w=353)]({{ site.baseurl }}/img/2013/03/fp-dci-11.png)

In the above example, there are two sites, each with two spine switches and two edge switches. The root for the multi-destination tree is on Spine-3 in Site B. For the hosts connected to the two edge switches in site A, broadcast traffic could follow the path from Edge-1 up to Spine-1, then over to Spine-3 in Site B, then to Spine-4, and then back down to the Spine-2 and Edge-2 switches in Site A before reaching the other host. Obviously there could be slightly different paths depending on topology, e.g. if the Spine switches are not directly interconnected. In future releases of NX-OS, the ability to create multiple FabricPath topologies will alleviate this issue to a certain extent, in that groups of 'local' VLANs can be constrained to a particular site, while allowing 'cross-site' VLANs across the DCI link.

**3: First Hop Routing localisation support is limited with FabricPath.**

When stretching L2 between sites, it's sometimes desirable to implement 'FHRP localisation' - this usually involves blocking HSRP using port ACLs or similar, so that hosts at each site use their local gateways rather than traversing the DCI link and being routed at the other site. The final point to be aware of is that when using FabricPath for layer 2 DCI, achieving FHRP localisation is slightly more difficult. On the Nexus 5500, FHRP localisation is supported using 'mismatched' HSRP passwords at each site (you can't use port ACLs for this purpose on the 5K). However, if you have any other FabricPath switches in your domain which aren't acting as a L3 gateway (e.g. at a third site), then that won't work and is not supported.

This is because FabricPath will send HSRP packets from the virtual MAC address at each site with the local switch ID as a source. Other FabricPath switches in the domain will see the same vMAC from two source switch IDs and will toggle between them, making the solution unusable. Also, bear in mind that FHRP localisation with FabricPath isn't (at the time of writing) supported on the Nexus 7000.

The issues noted above do not mean that FabricPath cannot be used as a method for extending layer 2 between sites. In some scenarios, it can be a viable alternative to the other DCI technologies as long as you are aware of the caveats above.

Hope this is useful - thanks for reading!
