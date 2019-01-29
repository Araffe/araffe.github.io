---
author: adraffe
comments: false
date: 2013-02-25 20:46:34+00:00
layout: post
link: http://adamraffe.com/2013/02/25/why-you-should-configure-fabricpath-vlans-everywhere/
slug: why-you-should-configure-fabricpath-vlans-everywhere
title: Why You Should Configure FabricPath VLANs Everywhere
wordpress_id: 153
tags:
- FabricPath
- Nexus
---

In a FabricPath deployment, it is important to have all FabricPath VLANs configured on **every** switch participating in the FP domain. Why is this? The answer lies in the way **multi-destination trees** are built.

A multi-destination tree is used to forward broadcast, unknown unicast and multicast traffic through the FabricPath network:

[![FP-MD-Tree]({{ site.baseurl }}/img/2013/02/fp-md-tree.png?w=550)]({{ site.baseurl }}/img/2013/02/fp-md-tree.png)

<!-- more -->In the above example, S1 is the root of this multi-destination tree, with the branches of the tree shown in red. Any multi-destination traffic (such as ARP requests) from say, S10 to S20 would take the path to S1 and then back down. So what does this have to do with configuring VLANs on every switch? Let's look at an example:

[![FP-MD-Tree2]({{ site.baseurl }}/img/2013/02/fp-md-tree21.png)]({{ site.baseurl }}/img/2013/02/fp-md-tree21.png)

In the above example, S10 and S20 are now configured to run vPC+ - this is commonly used between switch pairs to facilitate downstream switch connectivity into the FabricPath domain. We also have a VLAN (10) spanned between S10 and S20 for backup routing purposes (also common), with SVIs on both switches. However when we try and ping from S10 to S20 on VLAN 10, it fails. Why? The multi-destination tree in this case takes the path up to S1 and back down again - the Peer-Link between S10 and S20 is not part of this tree. What this means is that ARP requests will take the longer path up to S1 and back to S20 - **however VLAN 10 is not configured on S1 **so the ARP traffic is black holed. Adding VLAN 10 to S1 resolves the issue and traffic flows normally. Note that unicast traffic will take the direct path between S10 and S20 - it is only multi-destination traffic which uses the tree.

So the moral of this story is to configure your VLANs everywhere, even if you think traffic for a given VLAN might never use a particular switch - trying to prune manually by removing VLANs from certain switches will inevitably lead to your multi-destination tree breaking and traffic being black holed.
