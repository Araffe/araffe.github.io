---
author: adraffe
comments: false
date: 2013-02-27 20:59:26+00:00
layout: post
link: http://adamraffe.com/2013/02/27/nexus-1000v-layer-3-control-for-vsm-to-vem/
slug: nexus-1000v-layer-3-control-for-vsm-to-vem
title: 'Nexus 1000V: Layer 3 Control for VSM to VEM'
wordpress_id: 186
categories:
- Nexus 1000V
- Virtualisation
---

When the Nexus 1000V was first released, the only available control mode between VSM (Virtual Supervisor Module) and VEM (Virtual Ethernet Module) was layer 2 mode. This meant that the VSM and VEM had to be layer 2 adjacent (i.e. on the same VLAN). Layer 3 control mode was released a while ago however, which meant that the VSM and VEM could be on different VLANs / subnets. L3 control also makes things slightly simpler to set up as you don't need to worry about trunking control / packet VLANs everywhere and setting up port groups for these.

Assuming you want to use L3 control mode, there are a couple of decisions to make:<!-- more -->



	
  1. Which L3 interface you want to use on the VSM (control0 or mgmt0).

	
  2. Whether you use the mgmt VMK interface or set up a dedicated VMK interface for L3 control.


Let's look at the VSM end first. You have two choices: you can either use a dedicated control interface (control0), or you can use the existing mgmt0 interface (which gets set up when the N1KV is first installed and is also used for Telnet / SSH into the VSM). Either is fine - personally I have a slight preference to use a separate control0 interface and keep mgmt0 dedicated to management. You configure either control0 or mgmt0 under the 'svs-domain' config, as follows:

    
    svs-domain
     no control vlan
     no packet vlan
     svs mode L3 interface <strong>mgmt0 | control0</strong>


Note that you need to remove the control and packet VLAN configuration as they aren't relevant in L3 control mode.

The second choice you need to make is whether to use the existing management VMK interface, or whether a new one should be created for the VEM to VSM connectivity. As a refresher, a VMK interface is a virtual interface that the ESXi host itself uses to communicate with the outside world. Because we are trying to configure a layer 3 connection between the VSM and VEM, and because the Nexus 1000V is a layer 2 switch (therefore doesn't have any native L3 support built in), we need a VMK interface on the ESXi host to do this.

If you choose to use the management VMK interface (normally VMK0) for layer 3 control, that VMK will need to be moved over to the Nexus 1000V, where it will sit 'behind' the VEM, as follows:

[![L3-VSM-VEM](http://adamraffe.files.wordpress.com/2013/02/l3-vsm-vem.png)](http://adamraffe.files.wordpress.com/2013/02/l3-vsm-vem.png)

Bear in mind that the VSM will not 'see' the VEM (i.e. it won't appear in the output of 'sh mod') until the VMK interface is moved to the VEM. This is definitely the simpler of the two options and it cuts down on some extra admin as you don't have create a second VMK interface. However some people prefer having their ESXi management VMK interface 'out of band' - in other words, they prefer to have the management VMK interface remain on a standard vSwitch. This leads us to our second option:

[![L3-VSM-VEM2](http://adamraffe.files.wordpress.com/2013/02/l3-vsm-vem2.png)](http://adamraffe.files.wordpress.com/2013/02/l3-vsm-vem2.png)

This option keeps VMK0 (ESXi mgmt) on the standard vSwitch0, while a new VMK interface (VMK1) is created and used for L3 control between the VSM and VEM. If you choose this option, you may need to configure static routes on the ESXi host if the two VMK interfaces are in different VLANs - for example, a default gateway would be configured via VMK0, while a more specific static route would be configured via VMK1 towards the VSM IP address.
