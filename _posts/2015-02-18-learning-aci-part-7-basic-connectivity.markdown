---
author: adraffe
comments: true
date: 2015-02-18 12:48:47+00:00
layout: post
link: http://adamraffe.com/2015/02/18/learning-aci-part-7-basic-connectivity/
slug: learning-aci-part-7-basic-connectivity
title: 'Learning ACI - Part 7: Basic Connectivity'
wordpress_id: 873
categories:
- ACI
- Nexus 9000
tags:
- APIC
---

Welcome back! In this instalment, I'll look at how to get two bare metal hosts talking to each other in the fabric. In the [last post](http://adamraffe.com/2015/01/16/learning-aci-part-6-access-policies/), we talked about _access policies_. At the end of that post, we had created a number of policies and applied them to our switching nodes. If you recall, by doing that we had provisioned a range of VLANs on one or more ports on a leaf node, but we had not actually _enabled_ any VLANs on a port. In order to do that, we need to create at least one EPG and associate it with a port.

<!-- more -->

Here is the simple topology I'm going to use in this post:

[![Basic-Connectivity](https://adamraffe.files.wordpress.com/2015/02/basic-connectivity.jpg)](https://adamraffe.files.wordpress.com/2015/02/basic-connectivity.jpg)

In the above diagram, we have two hosts – host A and host B. Host A is connected to port E1/1 on leaf node 101, while host B is connected to port E1/1 on leaf node 102. We are going to place host A into the ‘Web’ EPG, with host B being placed into the ‘App’ EPG. Because these are bare metal hosts, the association of ports to EPGs will be done statically.

When we configured our access policies in part 6, we provisioned a pool of VLANs (500-550) on the ports that we wished to use on our leaf nodes. We are going to pick two VLANs from this pool – one for each of our bare metal connections – and use those VLANs as part of the static EPG mapping.

I’m making an assumption here that you already have a tenant, private network, bridge domain and subnet configured (as described in [part 5](http://adamraffe.com/2015/01/06/learning-aci-part-5-private-networks-bridge-domains-and-subnets/)). Assuming that has been done, let’s go ahead and create an application profile. Remember, an application profile is simply a container for EPGs and any associated contracts – app profiles are found under the ‘tenant’ tab:

[![Test-App](https://adamraffe.files.wordpress.com/2015/02/test-app.png)](https://adamraffe.files.wordpress.com/2015/02/test-app.png)

Here, we are creating an application profile called ‘Test-App’. The next step is to create our two EPGs – _Web_ and _App_. We do this under the application profile that we just created:

[![EPG-Create-1](https://adamraffe.files.wordpress.com/2015/02/epg-create-1.png)](https://adamraffe.files.wordpress.com/2015/02/epg-create-1.png)

Notice here that I am associating my EPG with a domain - in this case, the physical domain named 'Adam-Phys' that I created in part 6. Because we are connecting bare metal hosts to the fabric, we are going to select ‘create static path binding’. A static path binding is simply a way of manually specifying a switch port and VLAN that will be associated with our EPG. Because we have selected this option, an additional screen will become available to us where we must specify the port and VLAN – remember that the VLAN you choose here _must_ fall within the VLAN range that you enabled on the port during the access policy configuration:

[![Static-Path](https://adamraffe.files.wordpress.com/2015/02/static-path.png)](https://adamraffe.files.wordpress.com/2015/02/static-path.png)

We can now see the two EPGs that we have just configured:

[![EPGs](https://adamraffe.files.wordpress.com/2015/02/epgs.png)](https://adamraffe.files.wordpress.com/2015/02/epgs.png)

At this point, if everything has been configured properly, you should be able to ping the default gateway (the gateway address configured under the bridge domain) from both of your hosts. Note however at this point that no communication should be possible _between_ the two hosts – in order for that to happen we will need to configure some filters and contracts.

Let’s say we want to allow ICMP traffic between host A and host B. We’ll start off by configuring a filter named _icmp_ – this filter will simply specify ICMP traffic:

[![Filter](https://adamraffe.files.wordpress.com/2015/02/filter.png)](https://adamraffe.files.wordpress.com/2015/02/filter.png)

Now that we have a filter, the next step is to configure a contract – we’ll name this contract _web-to-app _and refer to the ‘icmp’ filter we created a second ago:

[![Contract](https://adamraffe.files.wordpress.com/2015/02/contract.png)](https://adamraffe.files.wordpress.com/2015/02/contract.png)

OK, so we have a filter and a contract – what we haven’t done yet however is apply these to any EPG. As you can see from the diagram at the beginning of this post, the Web EPG will _provide_ the web-to-app contract, while the App EPG will _consume_ the same contract. This creates a relationship between the two EPGs and will allow communication between them:

[![Provide-Consume](https://adamraffe.files.wordpress.com/2015/02/provide-consume.png)](https://adamraffe.files.wordpress.com/2015/02/provide-consume.png)

Once we have the contract in place, we should now be able to ping between our two hosts. Obviously this is a very basic example – it’s possible to use much more complex filter entries, with EPGs providing and consuming multiple contracts, service graphs, etc, but hopefully this post has given you an idea of how easy it is to set up application policies within ACI.

In the next post, I’ll take a look at how we can integrate ACI with the server virtualisation layer (specifically vSphere in this case) and how this makes things easier by automatically mapping EPGs to port groups and VLANs.

Thanks for reading!
