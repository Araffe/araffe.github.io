---
author: adraffe
comments: true
date: 2015-01-06 20:57:36+00:00
layout: post
link: http://adamraffe.com/2015/01/06/learning-aci-part-5-private-networks-bridge-domains-and-subnets/
slug: learning-aci-part-5-private-networks-bridge-domains-and-subnets
title: 'Learning ACI - Part 5: Private Networks, Bridge Domains and Subnets'
wordpress_id: 806
categories:
- ACI
- Nexus 9000
tags:
- APIC
---

Welcome to part 5 of this blog series - so far I have covered the following topics:



	
  * [Part 1](http://adamraffe.com/2014/12/03/learning-aci-part-1-overview/) contained a very brief overview of ACI and what the series would cover.

	
  * In [part 2](http://adamraffe.com/2014/12/03/learning-aci-part-2-bringing-up-a-fabric/), I talked through the fabric bring-up process.

	
  * [Part 3](http://adamraffe.com/2014/12/03/learning-aci-part-3-getting-familiar-with-the-apic/) was all about getting familiar with the APIC controller GUI.

	
  * In the last blog, [part 4](http://adamraffe.com/2015/01/02/learning-aci-part-4-application-profiles-epgs-contracts-and-filters/), we took a look at some of the most important policy constructs within ACI - application profiles, EPGs, contracts and filters.


Next on the list, we'll have a look at some networking concepts within ACI - namely private networks, bridge domains and subnets. Some of these are terms that you might not recognise, so what are they for?<!-- more -->

**Private Networks**

In ACI, a private network - sometimes known as a _context_ - is used to define a layer 3 forwarding domain within the fabric. A private network can support IP ranges which overlap with another private network on the fabric. If you're thinking you've seen this somewhere before, you're right - private networks look very similar to VRFs in the traditional networking world. In fact, a private network / context_ is _a VRF - if you go to a fabric enabled Nexus 9K switch, you can see that each private network you define is instantiated as a VRF. Generally speaking, each tenant will have at least one private network defined.

A private network is defined under a tenant, under the 'Networking' side menu:

[![Private-Networks](https://adamraffe.files.wordpress.com/2015/01/private-networks.png)](https://adamraffe.files.wordpress.com/2015/01/private-networks.png)

There are a number of policies you can associate with a private network, including OSPF and BGP timers, as well as how long end points should be retained. One interesting option within the private network configuration is the Enforced / Unenforced option. The default option here is 'Enforced' - this means that all security rules (contracts) associated with this private network will be enforced. It is however possible to disable this (Unenforced) which means that contracts will _not_ be enforced within this private network. This is like having an ACL with 'permit any' at the end of it, which of course means that inter-EPG communication will be allowed without the need for contracts.

**Bridge Domains**

Private networks are fairly to easy to understand, so what about bridge domains? A bridge domain is simply a layer 2 forwarding construct within the fabric, used to define a flood domain. You're probably now thinking "that's just like a VLAN" - and you'd be right, except that bridge domains are not subject to many of the same limitations as VLANs, such as the 4096 segment limit. In addition, bridge domains have (by default) a number of enhancements such as optimised ARP forwarding and unknown unicast flooding.

When you define a BD (from the same 'Networking' side menu we used to define our Private Network), a number of forwarding options are available if you select the 'custom' option from the drop down menu:

[![BD Options](https://adamraffe.files.wordpress.com/2015/01/bd-options.png)](https://adamraffe.files.wordpress.com/2015/01/bd-options.png)

Let's look at these options in more detail:



	
  * _L2 Unknown Unicast:_ This option controls whether the ACI spine proxy database will be used for unknown unicast traffic (the default), or whether 'traditional' flooding of unknown unicast traffic should occur. You would typically use 'flood' mode if extending the BD to an external network (such as to a legacy environment).

	
  * _Unknown Multicast Flooding:_ This controls whether traffic should be flooded to all ports on a leaf node, or whether it should be constrained to only ports where say a multicast receiver resides.

	
  * _ARP Flooding: _By default, ACI will convert ARP broadcast traffic into unicast traffic and send it to the correct leaf node. This option can be disabled if traditional ARP flooding is needed.

	
  * _Unicast Routing: _Unicast routing is enabled by default and is required if the fabric is performing routing for a BD (e.g. if a subnet is defined). You might wish to disable this if the fabric is not performing routing for a BD (i.e. a gateway resides outside the fabric, such as on a firewall or external router).


A bridge domain is always associated with a private network - you'll need to select one when you create the BD.

**Subnets**

When you create a bridge domain, you have the option of defining a _subnet _under the BD:

[![Subnet](https://adamraffe.files.wordpress.com/2015/01/subnet.png)](https://adamraffe.files.wordpress.com/2015/01/subnet.png)

When you define a subnet under a BD, you are creating an anycast gateway - that is, a gateway address for a subnet that potentially exists on every leaf node (if required). In the traditional world, think of this as an SVI interface that can exist on more than one node with the same address.

You'll also notice that we can control the scope of the subnet - private, public or shared. These are used as follows:



	
  * _Private: _A private subnet is one that is not advertised to any external entity via a L3 outside and will be constrained to the fabric.

	
  * _Public:_ A public subnet is flagged to be advertised to external entities via an L3 outside (e.g. via OSPF or BGP).

	
  * _Shared: _If a subnet is flagged as shared, it will be eligible for advertisement to other tenants or contexts within the fabric. This is analogous to VRF route leaking in traditional networks.


Hopefully this gives you a good overview of the main networking concepts within ACI - in part 6, I'm going to walk you through creating an _access policy _- this is a crucial part of attaching hosts and other devices to the fabric.
