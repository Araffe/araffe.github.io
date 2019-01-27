---
author: adraffe
comments: false
date: 2013-03-25 08:53:48+00:00
layout: post
link: http://adamraffe.com/2013/03/25/peer-gateway-doesnt-solve-l3-over-vpc/
slug: peer-gateway-doesnt-solve-l3-over-vpc
title: Peer-Gateway Doesn't Solve L3 Over vPC
wordpress_id: 452
categories:
- Nexus 7000
- Virtual Port Channel
tags:
- vPC
---

On a recent [Packet Pushers podcast](http://packetpushers.net/show-141-the-pace-of-change-is-picking-up-nfd5-discussion/), use of the Peer-Gateway feature on the Nexus 7000 and whether it resolves the lack of support for L3 over vPC was briefly discussed. The whole topic has been quite a big source of confusion, so let's answer it straight away: using Peer-Gateway to try and resolve L3 over vPC issues is not supported, but more importantly in most cases it _doesn't actually work_. The question is, why not? There are actually two reasons.<!-- more -->

**Issue #1: Data Plane**

The first issue is the one that many people who have implemented vPC know about. vPC on the Nexus 7000 has a built in rule stating that a packet received over the Peer-Link cannot be forwarded out onto another vPC. This is in place to prevent loops and duplicate packets, however it has a knock on effect on the ability to run routing to and from vPC connected devices, as shown in the following example.

[![Peer-Gateway-L3-1](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-1.png?w=550)](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-1.png)

In the above example, the layer 3 next hop (from the point of view of the router at the bottom) is Nexus 7000-2, however vPC is choosing to send the frame to Nexus 7000-1. As a result, the packet passes over the Peer-Link, where it is prevented from being sent out onto another vPC member port. It is therefore probable that approximately 50% of your traffic would be lost in this scenario. So if we now enable the Peer-Gateway feature, does that help?

Peer-Gateway was introduced to get around a specific problem found on certain NAS and load balancing platforms that have a habit of replying to traffic using the MAC address of the sending device, rather than the HSRP MAC. In non-vPC environments, that never caused an issue (in fact most people probably never even noticed it happening), but when using vPC, traffic gets lost for similar reasons as in the above example. Peer-Gateway fixes this by allowing a device to route packets originally intended for the other peer. So the question is, can't we enable Peer-Gateway to get around the L3 over vPC the above issue? The answer is yes, _for the data plane_. Unfortunately, the same feature has completely the wrong effect on the control plane for dynamic routing.

**Issue #2: Control Plane**

Let's now add a routing protocol - OSPF - to our example. We enable OSPF on both Nexus 7000 SVIs as well as the router at the bottom of our diagram with the intention of having adjacencies between all three. Peer-Gateway is enabled to get around the data plane issue described above.

[![Peer-Gateway-L3-2](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-2.png?w=550)](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-2.png)

In the above diagram, I am showing one OSPF adjacency between our router and Nexus 7000-2. In reality, there are two more adjacencies (one between the router and Nexus 7000-1, and one between the two Nexus 7Ks) but I'm not showing them here for clarity. In order to form those adjacencies, let's say that the OSPF packets between our router and Nexus 7000-2 take the alternate layer 2 path through Nexus 7000-1 as shown in the following diagram.

[![Peer-Gateway-L3-3](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-3.png?w=550)](http://adamraffe.files.wordpress.com/2013/03/peer-gateway-l3-3.png)

Now the important thing to remember is that we have Peer-Gateway enabled in the above topology. Peer-Gateway works by forcing routing to take place locally, which means that any OSPF packets which pass through Nexus 7000-1 are routed. What that also means is that the TTL will be decremented, and as most routing protocols use packets with a TTL of 1, the TTL is decremented to 0 - so the packet is dropped. The result of all this is that the OSPF adjacencies, in many cases, will never come up. Of course, you might be lucky and see all your packets being hashed to the 'correct' peer, however this is fairly unlikely.

So the bottom line is that enabling Peer-Gateway will solve one problem only to introduce another - the net result is that we cannot use this feature to support dynamic routing over vPC.
