---
author: adraffe
comments: true
date: 2015-10-18 02:58:43+00:00
layout: post
link: http://adamraffe.com/2015/10/18/learning-aci-part-11-transit-routing/
slug: learning-aci-part-11-transit-routing
title: 'Learning ACI - Part 11: Transit Routing'
wordpress_id: 1024
categories:
- ACI
- Nexus 9000
- SDN
tags:
- APIC
- External
- L3
- sdn
---

The 1.1(1j) & 11.1(1j) release of ACI introduced support for _transit routing._ Prior to this, the ACI fabric acted as a 'stub' routing domain; that is, it was not previously possible to advertise routing information from one routing domain to another through the fabric. I covered L3 Outsides in [part 9](http://adamraffe.com/2015/03/29/learning-aci-part-9-layer-3-external-connectivity/) of this series where I discussed how to configure a connection to a single routed domain. In this post, we'll look at a scenario where the fabric is configured with two L3 Outsides and how to advertise routes from one to another. Here is the setup I'm using:

[![Transit-Routing](https://adamraffe.files.wordpress.com/2015/10/transit-routing5.jpg)](https://adamraffe.files.wordpress.com/2015/10/transit-routing5.jpg)

<!-- more -->In my lab, I have a single 4900M switch which I have configured with two VRFs (Red and Green) to simulate the two routing domains. In the Red VRF, I have one loopback interface - Lo60 (10.1.60.60) which is being advertised into OSPF. In the Green VRF, I have Lo70 (10.1.70.70) and Lo90 (10.1.90.90) which are also being advertised into a separate OSPF process.

On the ACI fabric side, I have two L3 Outsides which correspond to the two VRFs. These two L3 Outsides are associated with a single private network (VRF) on the ACI fabric. OSPF is configured on both L3 Outsides with regular areas.

At this point, my OSPF adjacencies are formed and my ACI fabric is receiving routing information from both VRFs on the 4900M, as can be seen in the following output (taken from the 'Fabric | Inventory' tab and then under the specific leaf node, under the Protocols | OSPF section):

[![Screen Shot 2015-10-17 at 17.38.33](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-17-38-33.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-17-38-33.png)

You can see from this output that the fabric is receiving all of the routes for the loopback addresses configured under the Red and Green VRFs on the 4900M. Let's now take a look at the routing table for the Red VRF on the 4900M:

[![Screen Shot 2015-10-17 at 17.42.40](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-17-42-40.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-17-42-40.png)

It's clear from the above output that the Red VRF is not receiving information about either the 10.1.70.0 or 10.1.90.0 prefixes from the Green VRF - in other words, the ACI fabric is not currently re-advertising routes that it has received from one L3 Outside to the other.

Let's say I now want to advertise the prefixes from the Green VRF (10.1.70.0/24 and 10.1.90.0/24) into the Red VRF - how do I enable that? The main point of configuration for transit routing is found under 'External Networks' under the L3 / Routed Outside configuration. The key here is the _Subnets_ configuration - in previous versions of ACI, this has been used only to define the external EPG for policy control between inside and outside networks. Now however, the subnets configuration is used to also control transit routing in addition to policy control. Here are the options and what they are used for:



	
  * **Security Import Subnet: **Any subnet defined as a security import subnet is used for policy control. Effectively, a subnet defined in this way forms the external EPG - this is the same functionality that existed in previous ACI releases. If I define a subnet as a security import subnet, this subnet will be accessible from internal EPGs (or other external EPGs), as long as a suitable contract is in place. Importantly, this option has nothing whatsoever to do with the control of routing information into or out of the fabric.

	
  * **Export Route Control Subnet: **This option is used to control which specific transit routes are advertised out of the fabric. If I mark a subnet with export route control capability, I am telling the ACI fabric that I want those routes to be advertised to the external device. Note that this option controls the export of _transit routes only_ - it does _not_ control the export of internal routes configured on a bridge domain (you can see evidence of this in the 4900M routing table output above, in which you can already see one of my BD routes - 192.168.1.0/24 in the table).

	
  * **Import Route Control Subnet: **Similar to the export route control option, this option can be used to control which routes are allowed to be advertised into the fabric. Note that import route control is currently only available if BGP is used as the routing protocol.


How are these used in practice? Let's start with a simple example. I'm going to advertise just one of my 'Green' subnets (let's say 10.1.70.0/24) towards the 'Red' VRF. To do this, I add the 10.1.70.0 subnet to the L3 Out facing the Red VRF and mark it with the 'export route control' option:

[![Screen Shot 2015-10-17 at 19.33.01](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-33-01.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-33-01.png)

Now if I check the 'Red' routing table, I see the 10.1.70.0 prefix advertised from the fabric:

[![Screen Shot 2015-10-17 at 19.34.37](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-34-37.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-34-37.png)

If I add the 10.1.90.0/24 prefix to my subnets list, that transit route will also be advertised from the fabric to the Red VRF.

You might now be wondering how you would handle a large list of transit routes; would they all need to be individually entered into the subnets list? No - you can use the _Aggregate Export_ option. This option is currently only available when "0.0.0.0/0" is used as the subnet; essentially, this option tells the fabric to advertise _all_ transit routes. Checking the 'aggregate' option is important here - if you simply enter "0.0.0.0/0" as an export route control subnet without the aggregate option, the fabric will try and advertise the 0.0.0.0/0 route _only_. In my example, I've now removed the individual subnet and entered 0.0.0.0/0 with the aggregate option:

[![Screen Shot 2015-10-17 at 19.39.56](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-39-56.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-39-56.png)

I now see both my subnets advertised to the Red VRF:

[![Screen Shot 2015-10-17 at 19.41.12](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-41-12.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-41-12.png)

So that's the routing taken care of - but there's an additional step if you want traffic to flow. Remember, the ACI fabric models external destinations as EPGs. If you have two external destinations that need to communicate through the fabric, you must have both of those external destinations covered by a Security Import Subnet. As an example, if I wanted to allow hosts on 10.1.60.60 (part of the Red VRF) to talk to hosts on 10.1.70.70 (Green VRF), in addition to exporting the routes themselves (in both directions), I would need to define both of those subnets with the security import option:

**4900M-External Subnets Configuration:**

[![Screen Shot 2015-10-17 at 19.45.38](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-45-38.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-45-38.png)

**4900M-External-2 Subnets Configuration:**

[![Screen Shot 2015-10-17 at 19.47.57](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-47-57.png)](https://adamraffe.files.wordpress.com/2015/10/screen-shot-2015-10-17-at-19-47-57.png)

I would then need to provide / consume contracts between these networks for traffic to flow. In the above example, I could have just created a single 0.0.0.0/0 subnet on each side and marked it with export route control, aggregate export and security import options - that would effectively allow all routes and all destinations to communicate with each other (assuming contracts).

Thanks for reading!
