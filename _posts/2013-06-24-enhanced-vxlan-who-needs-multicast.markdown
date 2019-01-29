---
author: Adam
comments: false
date: 2013-06-24 17:23:07+00:00
layout: post
link: http://adamraffe.com/2013/06/24/enhanced-vxlan-who-needs-multicast/
slug: enhanced-vxlan-who-needs-multicast
title: 'Enhanced VXLAN: Who Needs Multicast?'
wordpress_id: 509
categories:
- Nexus 1000V
- VXLAN
---

VXLAN (Virtual eXtensible LAN) is an 'overlay' technology introduced a couple of years ago to solve the challenges associated with VLAN scalability and to enable the creation of a large number of logical networks (up to 16 million). It also has the benefit of allowing layer 2 segments to be extended across layer 3 boundaries due to the MAC-in-UDP encapsulation used.

One of the most significant barriers to the deployment of VXLAN is its reliance upon IP multicast in the underlying network for handling of broadcast / multicast / unknown unicast traffic - many organisations simply do not run multicast today and have no desire to enable it on their network. Another issue is that while VXLAN itself can support up to 16 million segments, it is not feasible to support such a huge number of multicast groups on any network - so in larger scale VXLAN environments, multiple VXLAN segments might have to be mapped to a single multicast group, which can lead to flooding of traffic where it is not needed or wanted.

With the release of version 4.2(1)SV2(2.1) of the Nexus 1000V, Cisco have introduced 'Enhanced VXLAN' - this augmentation to VXLAN removes the requirement to deploy IP multicast and can utilise the VSM (Virtual Supervisor Module) for handling the distribution of MAC addresses to each of the VEMs (Virtual Ethernet Modules). This post will delve into the specifics of enhanced VXLAN and explain how the various modes work.<!-- more -->

**Head End Replication
**

The first thing to understand about enhanced VXLAN is that there are two unicast modes in which the feature can run: "Flood and Learn" mode, or "MAC Distribution" mode (also known as "Floodless" mode). We'll discuss the differences between these modes in a bit, but the most important point about both of these is that for broadcast and multicast traffic, the VTEP (VXLAN Tunnel End Point) where the source VM resides will perform _head-end replication_ to the other VTEPs that have VMs residing in the same VXLAN / segment. This essentially means that the sending VTEP will replicate the broadcast or multicast packet locally and send it to all hosts participating in the VXLAN in question. Let's look at an example of this:

[![Enhanced-VXLAN-1]({{ site.baseurl }}/img/2013/05/enhanced-vxlan-12.png?w=550)]({{ site.baseurl }}/img/2013/05/enhanced-vxlan-12.png)

In the above example, all VMs reside in the same VXLAN. VM A sends a broadcast packet which reaches the local VEM / VTEP. In the original release of VXLAN, the VTEP would use the underlying IP multicast transport to ensure the broadcast packet reaches each of the VTEPs. With enhanced VXLAN, we are replicating the packet locally and sending a copy of the packet directly to the other VEMs / VTEPs. Note that if there were another host containing VMs not in our VXLAN, the broadcast packet would not be sent to that VTEP.

What about unknown unicast traffic? How we handle this type of traffic depends on which unicast mode we are operating in, which leads us to the next section.

**Unicast "Flood and Learn" Mode**

The first of the unicast modes is "Flood and Learn" mode; in this mode, we use head-end replication as described above to ensure that broadcast and multicast packets reach their destination, thus doing away with the requirement for IP multicast in the underlying transport network. However, in this mode we continue to learn MAC addresses via the "flood and learn" method - essentially nothing has changed in terms of how we learn MACs on our VEMs / VTEPs. For example:

[![Enhanced-VXLAN-2]({{ site.baseurl }}/img/2013/05/enhanced-vxlan-22.png)]({{ site.baseurl }}/img/2013/05/enhanced-vxlan-22.png)

When VTEP B receives the encapsulated VXLAN packet from VTEP A, it decapsulates it and learns that the address of VM A lives behind VTEP A (172.16.1.1). This is identical to how we learn MACs with the 'original' version of VXLAN.

What about unknown unicast traffic? In "Flood and Learn" mode, if a unicast packet needs to be switched but no entry is found in the MAC table, we will use head-end replication (just as we do for broadcast / multicast) to ensure that the packet reaches all the other VTEPs that contain VMs residing in the relevant VXLAN segment.

In 'non-enhanced' VXLAN, a VTEP would send out broadcast / multicast / unknown unicast frames using a specific multicast group in the transport network, safe in the knowledge that the frames would be delivered to every VTEP participating in the relevant VXLAN segment. So in unicast mode, how does a VTEP know where to send the locally replicated packets once they are encapsulated? In both of the unicast modes, the Nexus 1000V VSM keeps a record of all VTEPs operating within any given VXLAN segment and distributes this information to all the VEMs within the domain. We can see this on the VSM by running **show bridge-domain vteps**, e.g:

    
    N1KV-VXLAN# show bridge-domain test-vm vteps
    D: Designated VTEP     I:Forwarding Publish Incapable VTEP
    Bridge-domain: test-vm
     Ifindex     Module  VTEP-IP Address
     ------------------------------------------------------------------------------
     Veth3         3     172.16.1.1(D)
     Veth4         4     172.16.1.2(D)


Enabling Unicast Flood and Learn mode is simple: in the global configuration, use the command '**segment mode unicast-only'**.

But what if we don't want the traditional flood and learn behaviour in our VXLAN environment? Can't we use a control plane mechanism to distribute MAC address information throughout the domain? Look no further than the second unicast option in Enhanced VXLAN: MAC Distribution mode.

**Unicast "MAC Distribution" Mode (AKA "Floodless" Mode)**

In MAC Distribution mode, we still use head-end replication to deliver broadcast and multicast frames to the rest of the network. However in this mode we do not perform MAC learning based on data plane activity; instead we use the central control functionality of the Nexus 1000V - the Virtual Supervisor Module, or VSM - to keep track of all MAC addresses in the domain and disseminate that information to the VEMs / VTEPs on the system. Let's take a look at how this is achieved.

The first step in the learning process begins with all the VEMs / VTEPs becoming aware of the MAC addresses that are connected locally. This happens automatically when a virtual machine is powered up; locally connected MACs are installed in the VEM as 'static' entries. Once this has taken place, the next step is for the VEMs to report these statically installed MAC addresses to the VSM. We can check that this has taken place by using the command '**show bridge-domain mac**', as shown in the following example:

    
    N1KV-VXLAN(config)# sh bridge-domain test-vm mac
    Bridge-domain: test-vm
    MAC Address      Module     Port        VTEP-IP Address  VM-IP Address
    ------------------------------------------------------------------------------
    0050.5699.1bc3   3          Veth5       172.16.1.1       -
    0050.5699.3e45   4          Veth6       172.16.1.2       -


In the above output, we can see that the VSM is aware of two MAC addresses and is keeping track of which VEM / VTEP each one sits behind.

Finally, the VSM must distribute the MAC information to each of the VEMs / VTEPs - the VSM will ensure that the latest version of the MAC table is available to all VEMs. On the VEMs, the MAC entries that have been received from the VSM will be programmed as 'software installed'. It is possible to see these entries directly on the VEM modules using the following command:

    
    ~ # vemcmd show l2 bd-name test-vm
    Bridge domain    9 brtmax 4096, brtcnt 2, timeout 300
    Segment ID 5000, swbd 4096, "test-vm"
    Flags:  P - PVLAN  S - Secure  D - Drop
           Type         MAC Address   LTL   timeout   Flags    PVLAN    Remote IP
        SwInsta   00:50:56:99:3e:45   561         0                   172.16.1.2
         Static   00:50:56:99:1b:c3    51         0                      0.0.0.0


In the above output, the first entry in the list is a software installed MAC which has been sent to this VEM by the VSM. The second entry in the list is a locally connected MAC address (installed statically).

Remember earlier I said that unknown unicast traffic was handled differently depending on which unicast mode is in use? In MAC Distribution mode, it is assumed that if a MAC address exists on a VEM, it will be distributed to all VEMs through the MAC distribution mechanism. Because of this, there should be no unknown unicast MAC addresses anywhere within the VXLAN. What there _might_ be however, are unknown unicast addresses residing _outside_ of the VXLAN - in other words, accessible via a VXLAN gateway. So, if a MAC entry is not found in the table when a unicast packet is being switched, the local VEM / VTEP will send that packet only to any VTEPs designated as _VXLAN Gateways _(technically these are referred to as 'Forwarding Publish Incapable VTEPs) - it will not send the unknown unicast packet to every VTEP (as is the case in unicast "Flood and Learn" mode).

There are also plans to further enhance VXLAN using a BGP based control plane - this will allow VXLAN to scale out across two or more Nexus 1000V systems. This is being demonstrated at Cisco Live in Orlando this week - more details [here](http://blogs.cisco.com/datacenter/nexus-1000v-winning-multiple-awards-vxlan-scaling-with-bgp-and-more-on-display-at-cisco-live/).

Thanks for reading!


     Static   00:50:56:99:1b:c3    51         0                      0.0.0.0
