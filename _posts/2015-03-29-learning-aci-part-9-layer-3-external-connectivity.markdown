---
author: Adam
comments: true
date: 2015-03-29 18:12:07+00:00
layout: post
link: http://adamraffe.com/2015/03/29/learning-aci-part-9-layer-3-external-connectivity/
slug: learning-aci-part-9-layer-3-external-connectivity
title: 'Learning ACI - Part 9: Layer 3 External Connectivity'
wordpress_id: 924
categories:
- ACI
- Nexus 9000
tags:
- APIC
- External
- L3
---

So far in this series, everything we have discussed has been concerned with what happens inside the ACI fabric. At some point however, you will want to connect your fabric to the outside world, either at layer 2 or layer 3. In this part, we'll take a look at how to set up layer 3 connectivity from ACI to an external router, using a construct called the _Layer 3 Outside_.

Let's first take a look at the topology I'm going to discuss in this post:

[![L3-Outside]({{ site.baseurl }}/img/2015/03/l3-outside.jpg)]({{ site.baseurl }}/img/2015/03/l3-outside.jpg)<!-- more -->As you can see, we are going to set up a connection from the fabric using leaf node 101 (this will be our border leaf) to an external router - in this case, a Cisco 4900M. The physical connection between the leaf node and the 4900M will be a port-channel consisting of two interfaces. We will run OSPF between the fabric and our external router. Note also from the above diagram that MP-BGP will be running inside the fabric - this is needed for the distribution of external routing information through the fabric. The existence of MP-BGP is purely for this internal route distribution within the ACI fabric - it is _not_ running outside the fabric, e.g. there is no MP-BGP peering between the fabric and any external entity.

**Enable MP-BGP Inside the Fabric**

The first thing we need to do is enable MP-BGP to allow for the distribution of external routes inside the fabric. By default, BGP isn't enabled, but enabling it is a simple matter. From the main **Fabric** tab, select **Fabric Policies**, drill down to **Pod Policies** and then under Policies, you'll find an item entitled _BGP Route Reflector Default_. Select this item and on the right hand side of the screen, you'll see there are two things to configure: AS number and Route Reflector Nodes.

First, choose an AS number (bear in mind that you need to match the AS number outside of your fabric if you intend to use iBGP between the fabric and external routers). You can then choose which spine nodes you want to act as route reflectors inside the fabric - in the example below I am using both of the spine nodes in my network as route reflectors (203 and 204):

[![BGP-RR]({{ site.baseurl }}/img/2015/03/bgp-rr.png)]({{ site.baseurl }}/img/2015/03/bgp-rr.png)

That's all we need to do to enable BGP to run - you can check that this is the case from the **Inventory** tab under the **Fabric** menu. Select one of your spine nodes and then drill down to Protocols and then BGP - you should see some sessions formed to the lead nodes in your network:

[![BGP Sessions]({{ site.baseurl }}/img/2015/03/bgp-sessions.png)]({{ site.baseurl }}/img/2015/03/bgp-sessions.png)

**Configure Access Policies
**

Now that we have MP-BGP enabled inside the fabric, the next step is to configure any access policies required. In this post, I'm going to use a VLAN and SVI combination to build the layer 3 connection between the fabric and my external router. I'll therefore need to configure a VLAN pool, Attachable Entity Profile, Interface Policy Group, etc and apply those to the interfaces I will use for the external L3 connection.

I've already covered configuration of access policies in [part 6](http://adamraffe.com/2015/01/16/learning-aci-part-6-access-policies/) so I won't cover that again here, but one key difference here is that rather than configuring a physical domain (as shown in part 6), I now need to use an _External Routed Domain_ and apply my VLAN pool to that domain instead.

**Configure the External Routed Network
**

Before we go any further, let's take a look at what I am trying to achieve logically:

[![L3-Outside-Logical]({{ site.baseurl }}/img/2015/03/l3-outside-logical.jpg)]({{ site.baseurl }}/img/2015/03/l3-outside-logical.jpg)

I have a single virtual machine residing in an End Point Group (EPG) named 'Web'. This EPG is associated with a bridge domain, also named Web - the BD has a gateway of 192.168.1.1 configured which the VM can ping. I am going to create a layer 3 outside to allow OSPF routing to the 4900M external switch - finally, I am going to configure a contract between my internal 'Web' EPG and the external destinations I am trying to reach.

We can now switch to the tenant view and begin to configure the L3 Outside - under the **Networking** menu underneath the tenant, you'll find **External Routed Networks**. Right click on this item and select **Create Routed Outside**. In the dialog box you are presented with, you'll need to name the L3 Outside, select BGP or OSPF as the routing protocol (or neither if you want to use static routing), select the private network (context) you want to associate this L3 Outside to and select the external routed domain you configured during the access policy configuration in the last section:

[![L3-Outside-Main]({{ site.baseurl }}/img/2015/03/l3-outside-main.png)]({{ site.baseurl }}/img/2015/03/l3-outside-main.png)

In the above example, I am naming my L3 Outside '4900m-External'. I have chosen OSPF as the routing protocol with area 10 and associated this L3 Outside to a private network named 'adam-net1'. I have also associated this L3 Outside with an external routing domain named 'PhyD_L3_out' which I created earlier.

An important point here is that the ACI fabric (at the time of writing) supports OSPF running as an _NSSA_ (not so stubby area) only.

The next step is to configure nodes and interfaces. First, click the plus sign under 'Nodes and Interfaces Protocol Profiles', configure a name for your node profile and and then the plus sign next to 'Nodes'. From here, you can select the leaf node that you are using for the L3 Outside (in my case I am using Node-101) and configure a router ID for the node:

[![Configure Node]({{ site.baseurl }}/img/2015/03/configure-node.png)]({{ site.baseurl }}/img/2015/03/configure-node.png)

After submitting, add an OSPF interface profile. From here, you can select the interface type and interfaces which you want to use for this connection. Note here that there are three types of interface available: _Routed Interfaces, SVIs, _or _Routed Sub-Interfaces._ I'm going to use the SVI option here, which means that I will have a VLAN between my fabric and external router:

[![OSPF -Interface]({{ site.baseurl }}/img/2015/03/ospf-interface.png)]({{ site.baseurl }}/img/2015/03/ospf-interface.png)

In my example, I am using a port-channel for the physical connection between my fabric and external router - I've selected VLAN 12 as the encapsulation (this must fall within the range configured in the VLAN pool you are using!). Finally, I've configured an IP address of 172.16.1.1/24 for the SVI on this border node. One important point here is that this SVI is _not the same as an anycast gateway configured as part of a regular bridge domain_. This is a separate SVI interface used solely for external routing from the border leaf node. Submit this configuration and go back to the main interface profile page. We now need to create an OSPF interface policy using the drop down box in the middle of the page:

[![OSPF-Int-Pol]({{ site.baseurl }}/img/2015/03/ospf-int-pol.png)]({{ site.baseurl }}/img/2015/03/ospf-int-pol.png)

I've named my OSPF interface policy '4900M-OSPF-Int' and selected broadcast as a network type. One important thing you must do here is tick the 'advertise subnet' check box - if you don't tick this, nothing will be advertised from your L3 Outside! Once you've completed this, you can submit your interface profile.

That's it for the interface and node configuration - the next step is to configure external networks. An external network (sometimes referred to as an 'external EPG') is simply an external destination that we are trying to reach from within the fabric; in my example I'm configuring 10.1.0.0/16 as my external destination. Note that it is possible to use 0.0.0.0/0 if you want to define any address as a destination in the external network.

[![external-networks]({{ site.baseurl }}/img/2015/03/external-networks.png)]({{ site.baseurl }}/img/2015/03/external-networks.png)

At this point, I am going to configure my external router with some basic OSPF config - I'll add an SVI on the 4900M for VLAN 12, configure an IP address of 172.16.1.2/24 and add the network to OSPF.

    
    interface Vlan12  
     ip address 172.16.1.2 255.255.255.0 
    ! 
    router ospf 1 
     area 10 nssa 
     router-id 5.5.5.5 
     network 172.16.1.0 0.0.0.255 area 10


I also have a number of loopback interfaces configured as part of area 0.

Let's now take a look at our fabric to see if any OSPF adjacencies are up. From the **Fabric **tab, I choose **Inventory** and then drill down to the leaf node in question (in my case node 101). Under **Protocols** and then **OSPF**, I can choose to view OSPF neighbours, interfaces, routes and so on. If I look at under Neighbors, I see my 4900M external router as an adjacency:

[![OSPF-adjacency]({{ site.baseurl }}/img/2015/03/ospf-adjacency.png)]({{ site.baseurl }}/img/2015/03/ospf-adjacency.png)

Note that VLAN 88 has been automatically chosen by the fabric as the SVI interface on the leaf node. If I now look at the 'Routes' option, I can see that my external router is advertising a number of prefixes to my fabric:

[![OSPF-Routes]({{ site.baseurl }}/img/2015/03/ospf-routes.png)]({{ site.baseurl }}/img/2015/03/ospf-routes.png)

Let's now check on my external router to see whether any routes have been advertised from my fabric:

    
    DCN-4900M-2#sh ip route
    Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
           D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
           N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
           E1 - OSPF external type 1, E2 - OSPF external type 2
           i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
           ia - IS-IS inter area, * - candidate default, U - per-user static route
           o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
           + - replicated route, % - next hop override
    
    Gateway of last resort is not set
    
          10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
    C        10.1.60.60/32 is directly connected, Loopback60
    C        10.1.60.61/32 is directly connected, Loopback61
    C        10.1.60.62/32 is directly connected, Loopback62
    C        10.1.60.63/32 is directly connected, Loopback63
    C        10.1.60.64/32 is directly connected, Loopback64
    C        10.1.60.65/32 is directly connected, Loopback65
    C        10.1.123.8/29 is directly connected, Vlan31
    L        10.1.123.12/32 is directly connected, Vlan31
    C        10.1.123.136/29 is directly connected, Vlan41
    L        10.1.123.140/32 is directly connected, Vlan41
          172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
    C        172.16.1.0/24 is directly connected, Vlan12
    L        172.16.1.2/32 is directly connected, Vlan12


Hmm, nothing seems to be advertised from the fabric to the 4900M - why is this? There are a couple of settings we need to configure on the bridge domain in order for subnets to be advertised. First of all, I need to flag the internal subnets I want to advertise as _Public_. This setting is found within the subnet configuration under the bridge domain:

[![Subnet-Public]({{ site.baseurl }}/img/2015/03/subnet-public.png)]({{ site.baseurl }}/img/2015/03/subnet-public.png)

Secondly, I need to associate my bridge domain with the L3 Outside. If I click on my bridge domain, there is a box named 'Associated L3 Outs':

[![Associated-L3-Outs]({{ site.baseurl }}/img/2015/03/associated-l3-outs.png)]({{ site.baseurl }}/img/2015/03/associated-l3-outs.png)

Once I have associated my L3 Outside to my bridge domain, the subnet in question (192.168.1.0/24) is advertised to my external router:

    
    DCN-4900M-2#sh ip route
    Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
           D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
           N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
           E1 - OSPF external type 1, E2 - OSPF external type 2
           i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
           ia - IS-IS inter area, * - candidate default, U - per-user static route
           o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
           + - replicated route, % - next hop override
    
    Gateway of last resort is not set
    
          10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
    C        10.1.60.60/32 is directly connected, Loopback60
    C        10.1.60.61/32 is directly connected, Loopback61
    C        10.1.60.62/32 is directly connected, Loopback62
    C        10.1.60.63/32 is directly connected, Loopback63
    C        10.1.60.64/32 is directly connected, Loopback64
    C        10.1.60.65/32 is directly connected, Loopback65
    C        10.1.123.8/29 is directly connected, Vlan31
    L        10.1.123.12/32 is directly connected, Vlan31
    C        10.1.123.136/29 is directly connected, Vlan41
    L        10.1.123.140/32 is directly connected, Vlan41
          172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
    C        172.16.1.0/24 is directly connected, Vlan12
    L        172.16.1.2/32 is directly connected, Vlan12
    <strong>O N2  192.168.1.0/24 [110/20] via 172.16.1.1, 00:00:03, Vlan12</strong>


So at this point, we should be able to ping between internal and external networks, right? Actually we can't:

    
    DCN-4900M-2#ping ip 192.168.1.2 source loopback60 
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
    Packet sent with a source address of 10.1.60.60 
    .....
    Success rate is 0 percent (0/5)


The reason we still have no connectivity is that I do not yet a _contract_ in place between my internal EPG (Web) and my external network (10.1.0.0/16). I can add this by clicking on the external network (external EPG) I created during the L3 Outside configuration:

[![Provided-Consumed]({{ site.baseurl }}/img/2015/03/provided-consumed.png)]({{ site.baseurl }}/img/2015/03/provided-consumed.png)

I've simply provided the 'default' contract (which permits anything) from the external network. I'll consume the same contract from my internal 'Web' EPG, which results in the following:

[![Provided-Consumed2]({{ site.baseurl }}/img/2015/03/provided-consumed2.png)]({{ site.baseurl }}/img/2015/03/provided-consumed2.png)

Let's try initiating a ping again from the external router:

    
    DCN-4900M-2#ping ip 192.168.1.2 source loopback60
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
    Packet sent with a source address of 10.1.60.60 
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms


So now that we have the contracts in place, we have full connectivity between our internal and external network.

Thanks for reading!
