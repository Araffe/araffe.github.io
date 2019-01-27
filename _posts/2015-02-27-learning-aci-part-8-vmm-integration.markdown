---
author: Adam
comments: true
date: 2015-02-27 23:30:02+00:00
layout: post
link: http://adamraffe.com/2015/02/27/learning-aci-part-8-vmm-integration/
slug: learning-aci-part-8-vmm-integration
title: 'Learning ACI - Part 8: VMM Integration'
wordpress_id: 897
categories:
- ACI
- Nexus 9000
tags:
- APIC
- VMM
---

Welcome to part 8 - let's quickly recap what we have covered so far in this series:



	
  * [Part 1](http://adamraffe.com/2014/12/03/learning-aci-part-1-overview/) introduced this series and discussed what topics would be covered, as well as a very brief overview of ACI.

	
  * In [part 2](http://adamraffe.com/2014/12/03/learning-aci-part-2-bringing-up-a-fabric/), I took a look at the fabric bring up process

	
  * Next, we took a [tour through the APIC GUI](http://adamraffe.com/2014/12/03/learning-aci-part-3-getting-familiar-with-the-apic/) to get us familiar with the interface.

	
  * [Part 4](http://adamraffe.com/2015/01/02/learning-aci-part-4-application-profiles-epgs-contracts-and-filters/) looked at some of the most important ACI constructs - app profiles, EPGs, contracts and filters.

	
  * We had a look at networking concepts in ACI in [part 5](http://adamraffe.com/2015/01/06/learning-aci-part-5-private-networks-bridge-domains-and-subnets/).

	
  * In [part 6](http://adamraffe.com/2015/01/16/learning-aci-part-6-access-policies/), we discussed access policies and how they are used to provision ports.

	
  * Last time out in [part 7](http://adamraffe.com/2015/02/18/learning-aci-part-7-basic-connectivity/), I walked through setting up basic connectivity between two bare metal hosts.


OK, so what's next? In this part, we'll discuss _VMM Integration_. What does this mean exactly? Firstly, VMM stands for _Virtual Machine Manager_ - in other words, we are talking about integration with a VM management system such as VMware vCenter, Microsoft SCVMM and so on. At the time of writing this post, ACI supports integration with vCenter (others will be supported later), so this is what we'll concentrate on here. I should also point out that we could also use the Cisco _Application Virtual Switch_ (AVS) to achieve this integration, but I'm going to focus on using the regular VMware distributed virtual switch in this post.<!-- more -->

But what exactly do we gain by integrating ACI with a VM management system? Why would we actually want to do this? Consider a simple example in a 'traditional' environment: let's say a server virtualisation administrator is provisioning a set of virtual machines to be used for a certain application. These VMs will be used across a number of application tiers (let's say web, app and DB just for simplicity). We can assume that there will be a port group at the virtual switch layer corresponding to each application tier and that each one will presumably map back to a VLAN on the physical network. But which VLAN should be used for each port group? The virtualisation administrator will need to work together with the network administrator to agree on which VLAN should be used for each application tier / port group. Is this really something a virtualisation admin wants to be worrying about? For that matter, does the network admin want to be worried about which VLAN is used for any given application tier? The answer in most cases is "no", which is precisely where ACI VMM integration comes in.

So how does this work? Fundamentally, configuring VMM integration from ACI results in a connection being made between the APIC and the vCenter server, at which time an "APIC controlled" distributed virtual switch is automatically created at the vSphere level. There is nothing particularly special about this DVS - the only difference from a 'normal' DVS is that this one is automatically created by the APIC and controlled by ACI. To understand what this allows us to do, take a look at the following drawing:

[![VMM-Integration](https://adamraffe.files.wordpress.com/2015/02/vmm-integration1.jpg)](https://adamraffe.files.wordpress.com/2015/02/vmm-integration1.jpg)

In this example, we have created a tenant on the ACI fabric called _IT_. Within the IT tenant, we have an application profile named _App1_. Within this app profile, we are creating three End Point Groups (EPGs) corresponding to our three-tiered application (Web, App, DB). Now this is where it gets interesting - as we created our EPGs, the APIC automatically created a corresponding port group on the DVS. Note the naming of the port group as _<tenant>|<app-profile>|<EPG-name>_, for example _IT|App1|Web_. You might also notice that each of our port groups has been allocated a VLAN (e.g.  Web = VLAN 100, App = VLAN 101, etc). These VLANs have been automatically allocated to our port groups by the APIC - they have been taken from a dynamic VLAN pool which we have pre-configured on the APIC.

What does the virtualisation administrator have to do here? He or she simply adds the VMs to the correct port groups - no need to worry about provisioning these port groups, or which VLAN should be associated with each one - it all happens automatically.

Now that we understand what VMM integration provides, how do we configure it? For this to work, there are two things that must happen: firstly, the APIC obviously needs to communicate with the vCenter; secondly, the APIC must learn the location of the ESXi host (i.e. which switch / port it is connected to) using either CDP or LLDP.

Let's start by setting up the VMM domain. The first thing we do is create a _dynamic VLAN pool_. You may remember in [part 6](http://adamraffe.com/2015/01/16/learning-aci-part-6-access-policies/), we set up a static VLAN pool for bare metal host connectivity. In that case, VLANs were allocated manually by the administrator - in this case, we want VLANs to be allocated automatically upon EPG creation, hence the dynamic nature of the pool. In the following example, I am creating a dynamic VLAN pool named "CL-Dynamic", with VLANs in the range 200 - 300:

[![Dynamic-VLANs](https://adamraffe.files.wordpress.com/2015/02/dynamic-vlans.png)](https://adamraffe.files.wordpress.com/2015/02/dynamic-vlans.png)

The next step is to create the VMM domain itself. To do this, you need the credentials and IP address / hostname of the vCenter - notice that you also reference the dynamic VLAN pool created in the last step:

[![Create-VMM-Domain](https://adamraffe.files.wordpress.com/2015/02/create-vmm-domain.png)](https://adamraffe.files.wordpress.com/2015/02/create-vmm-domain.png)

If all is well, you should now see an active connection to the vCenter (from the 'Inventory' tab under VM Networking). You should see details about the vCenter version as well as some info about the ESXi hosts:

[![VMM-Domain](https://adamraffe.files.wordpress.com/2015/02/vmm-domain.png)](https://adamraffe.files.wordpress.com/2015/02/vmm-domain.png)

One piece of manual configuration you will need to perform here is to add the physical NICs on the ESXi host (vmnics) to the new DVS that was just created.

At this point, you need to create an attachable entity profile and interface policy - the AEP should reference the VMM domain we just created. When creating the interface policy, be sure to enable CDP or LLDP as this is needed to discover the location of the ESXi host. Finally, associate the interface policy to the relevant switches and ports. Refer back to [part 6](http://adamraffe.com/2015/01/16/learning-aci-part-6-access-policies/) for more guidance on this process.

If you have configured everything correctly, checking the relevant vmnic under the ESXi hosts in the VM Networking Inventory tab in the APIC should display the leaf port to which the host is connected:

[![VMM-LLDP](https://adamraffe.files.wordpress.com/2015/02/vmm-lldp.png)](https://adamraffe.files.wordpress.com/2015/02/vmm-lldp.png)

In the above example, we can see that vmnic2 on this particular ESXi host is learning that it is connected to leaf node 101 on port 1/31 via LLDP. Note that this step is important - if you don't see the LLDP / CDP information under the correct vmnic, the APIC will not know which ports to apply policy to and connectivity will ultimately fail.

The final step is to actually to create the EPG - recall in part 7, we associated our EPG with a _physical domain_ - we follow a similar process here, but this time we are associating our EPG with the _virtual domain _we just created:

[![Domain-Add](https://adamraffe.files.wordpress.com/2015/02/domain-add1.png)](https://adamraffe.files.wordpress.com/2015/02/domain-add1.png)

Once we have done this, we should see the port group that corresponds to our EPG appear on the DVS:

[![vCenter-view](https://adamraffe.files.wordpress.com/2015/02/vcenter-view.png)](https://adamraffe.files.wordpress.com/2015/02/vcenter-view.png)

Here, you can see that I have an EPG named 'CiscoLiveEPG' as part of an application profile called 'CiscoLiveApp', all contained under a tenant named 'adraffe'. The APIC has created a port group with the corresponding name and has used a VLAN from the dynamic pool that we created originally.

All that remains now is for the virtualisation administrator to attach the VM to this port group and you are done!

Hopefully this has given you a good overview on VMM integration - this is a cool feature which can really reduce the burden on both network and server admins. Thanks for reading.
