---
author: adraffe
comments: false
date: 2013-05-22 20:05:32+00:00
layout: post
link: http://adamraffe.com/2013/05/22/single-homed-vs-dual-homed-fabric-extenders/
slug: single-homed-vs-dual-homed-fabric-extenders
title: Single-Homed vs Dual-Homed Fabric Extenders
wordpress_id: 586
categories:
- Nexus 5000
- Nexus 7000
tags:
- FEX
- Nexus 2000
---

A question that I still get quite a lot is how to connect Nexus 2000 Fabric Extenders to their parent switches - is it better to single attach them to the parent, or to dual-home the FEX to both parent switches (using vPC)? Of course there is no right answer for every situation - it depends on the individual environment and sometimes personal preference, but here are a few of my thoughts on this.<!-- more -->

**We can dual home the FEX to the parent Nexus 5500 - that must be better, right?**

This is a fairly common line of thinking - many people assume dual homing is automatically better than single homing, but that's not necessarily always the case. The best question to ask is what failure scenario you are trying to protect against, and where the recovery will happen (within the network or at the host level). For example, if you are trying to guard against the failure of a single Nexus 2000 Fabric Extender, then dual homing that Fabric Extender to two parent switches will clearly not help you - you need to ensure that hosts / servers are dual homed to two Fabric Extenders to mitigate this risk.

Dual-homing Fabric Extenders using vPC does give you some protection against the failure of a single Nexus 5000 parent switch - if that happens, the FEX still has connectivity to the second parent 5K, so the host ports remain up and hosts should not see any interruption. This is particularly helpful in a situation where there are single attached hosts - these hosts have no other protection against the failure of a parent switch, so this is one situation where dual-homing of FEXs might make sense. Although it is stating the obvious somewhat, it's worth pointing out here that if you have single attached hosts there is still a single point of failure (i.e. the FEX itself could fail), so it's always a better idea to dual-home hosts to the network in the first place.

**Why would I _not_ want to dual-home my Fabric Extenders?**

Dual-homing a Nexus 2000 to its parent switches brings some challenges compared to single attaching. Firstly, it is necessary to configure host ports residing on the FEX from _both_ parent switches - in other words, the configuration for the ports in question needs to be kept synchronised across the two parent switches. Of course you can do this manually, or use the config sync feature but in general there is some management overhead with maintaining this. Another downside is that you effectively cut the number of FEXs you can support by half when you dual-home.

**What about connecting Fabric Extenders to the Nexus 7000?**

If you are connecting Nexus 2000s to Nexus 7000 parent switches, then you have no choice (at the time of writing, May 2013) - dual-homing of FEXs isn't currently supported. I've heard some people complain about the lack of support for this on the 7K, but in reality it goes back to what you are trying to protect against. Given that we have already established that dual-homing of a FEX helps to protect against a failure of one parent switch, how often do you expect to lose an entire Nexus 7000 (considering it usually has redundant Supervisors, Fabrics, Linecards, etc)? That should (hopefully) be quite a rare event - but if the worst does happen and you lose one of your 7Ks, as long as your hosts are dual-homed, they should fail over to the remaining 7K / 2K using whatever teaming / port-channelling method is in place. Support for dual-homing FEXs to Nexus 7000s may come in the future, but my opinion is that not having it today is no great loss.

The bottom line is that dual-homing of Fabric Extenders to Nexus 5000 parent switches _might_ give you some benefit in a scenario where it is not possible to dual attach hosts / servers to the network - however I would question the importance of those servers if that is the case. If you dual attach all your hosts to two Fabric Extenders (which you should), then FEX dual-homing gives you quite a limited benefit in my view.
