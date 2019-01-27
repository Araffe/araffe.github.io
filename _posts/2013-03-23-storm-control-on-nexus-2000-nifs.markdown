---
author: adraffe
comments: false
date: 2013-03-23 19:02:47+00:00
layout: post
link: http://adamraffe.com/2013/03/23/storm-control-on-nexus-2000-nifs/
slug: storm-control-on-nexus-2000-nifs
title: Storm Control on Nexus 2000 NIFs
wordpress_id: 427
categories:
- Nexus 5000
---

NX-OS release 5.2(1)N1(2) added support for storm control on Nexus 2000 NIFs / FEX Fabric Interfaces (this is also available on 6.0(2)N2(1) for the Nexus 6000) - these are the interfaces used to connect the parent Nexus 5500 or 6000 to the Fabric Extender. I looked into this feature recently for a customer so thought a quick overview might be useful as there are a couple of things to be aware of.

Firstly, the storm control percentage value that you configure gets implemented as a percentage of the total speed of the port-channel between 5K / 6K and FEX. Here's an example:

[![NIF-SC-1](http://adamraffe.files.wordpress.com/2013/03/nif-sc-1.png)](http://adamraffe.files.wordpress.com/2013/03/nif-sc-1.png)

<!-- more -->In the above drawing, we have four 10GE interfaces used as FEX Fabric ports (i.e. the links between the 5K and 2K). All four links within this port-channel on the 5K are on the same port ASIC (this matters!!), and broadcast storm control is configured on the FEX Fabric port-channel, with a value of 50%. So in this case, a total of 20Gbps of broadcast traffic would be allowed into the parent Nexus 5500 before the threshold is reached and we start dropping traffic. This is pretty straightforward and what you would expect to see.

I said in the example above that the port ASIC allocations matter - this is because the behaviour is slightly different if the FEX Fabric port-channel is spread across multiple port ASICs on the Nexus 5K. Here's another example:

[![NIF-SC-2](http://adamraffe.files.wordpress.com/2013/03/nif-sc-2.png)](http://adamraffe.files.wordpress.com/2013/03/nif-sc-2.png)

In the second example, two of the member links in the FEX Fabric port-channel are connected to ports serviced by the first UPC (Unified Port Controller) ASIC. The other two ports in the channel are using a different UPC ASIC. The storm control percentage is still calculated based on the total bandwidth in the port-channel (10%, or 4Gbps in this case) - however, importantly, each UPC port ASIC_ enforces the threshold independently_. What that means in practice is that each UPC will allow 4Gbps of broadcast traffic, for a total of 8Gbps into the switch. If you aren't aware of this then you could end up with more traffic than you expect before the switch takes action.

Note that the above behaviour is applicable to normal port-channels as well (not just FEX NIF ports).
