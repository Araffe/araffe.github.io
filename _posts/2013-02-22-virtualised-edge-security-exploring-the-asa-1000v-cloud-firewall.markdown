---
author: adraffe
comments: false
date: 2013-02-22 15:48:38+00:00
layout: post
link: http://adamraffe.com/2013/02/22/virtualised-edge-security-exploring-the-asa-1000v-cloud-firewall/
slug: virtualised-edge-security-exploring-the-asa-1000v-cloud-firewall
title: 'Virtualised Edge Security: Exploring the ASA 1000V Cloud Firewall'
wordpress_id: 8
categories:
- Security
- Virtualisation
---

What is the ASA 1000V? It is a virtualised edge firewall that runs in conjunction with the Nexus 1000V switch. The ASA 1000V runs as a virtual machine, and provides a secure default gateway for other VMs in the environment. Many of the features from the physical ASAs are supported, such as NAT, failover and site-to-site IPSec VPNs, however there are a few features which are not supported in the current release such as IPv6, multiple contexts, dynamic routing and transparent mode firewalling.

An ASA 1000V has four "physical" interfaces - 'Inside', 'Outside', 'Management' and 'Failover':

[![ASA1K-interfaces](http://adamraffe.files.wordpress.com/2013/02/asa1k-interfaces1.jpg)](http://adamraffe.files.wordpress.com/2013/02/asa1k-interfaces1.jpg)

<!-- more -->Of course, these aren't really physical interfaces; they are virtual interfaces residing on a VM, and each of these interfaces maps directly to a port-profile that needs to be configured on the Nexus 1000V before the ASA is installed. To refresh your memory, a port profile is a collection of interface level commands on the Nexus 1000V which can be applied to one or more virtual Ethernet interfaces connecting to a VM. Logically, the ASA 1000V virtual machine is sitting 'behind' the Nexus 1000V VEM, and the mapping of port-profiles to ASA 1000V interfaces looks like this:

[![ASA1K-Port-Profiles](http://adamraffe.files.wordpress.com/2013/02/asa1k-port-profiles1.jpg?w=710)](http://adamraffe.files.wordpress.com/2013/02/asa1k-port-profiles1.jpg)

An important point to remember about the ASA 1000V is that in the initial release - 8.7(1) - only two data interfaces are supported (Inside and Outside). One of these interfaces (the Inside interface) must be designated as the 'Service' interface - this interface sends and receives vPath tagged traffic to and from the Nexus 1000V. I will (hopefully) explore vPath in more detail in a future post, but essentially this is a mechanism which allows the Nexus 1000V to intercept traffic and redirect it to a virtual service (which in our case is the ASA 1000V).

So how does the ASA 1000V actually protect virtual machines? For this, we need to introduce the concept of a 'security profile'. A security profile is essentially a collection of security policies - the difference here is that the security profile gets assigned to a set of protected VMs through the use of Nexus 1000V port profiles. The following diagram shows how this looks:

[![ASA1K-Security-Profiles](http://adamraffe.files.wordpress.com/2013/02/asa1k-security-profiles2.jpg)](http://adamraffe.files.wordpress.com/2013/02/asa1k-security-profiles2.jpg)

A few things are going on here:

1) A security-profile named 'Web' has been created on the ASA 1000V that contains the policy information to be applied to the protected VMs. This security-profile is typically created using VNMC (Virtual Network Management Centre) or through the ASDM GUI.

2) The security-profile 'Web' has been assigned to an interface on the ASA 1000V, named 'security-profile1'.

3) On the Nexus 1000V, the port-profile to which the protected VMs will be assigned (in this case named 'web-vm') has some additional configuration that ties the ASA 1000V security-profile to it.

4) The protected VMs use the 'Inside' interface of the ASA 1000V as their default gateway.

One slight area of confusion here is that although the 'Inside' interface of the ASA 1000V is on the same VLAN (VLAN 10 in this case) as the vNICs of the protected VMs, the Inside interface sits in a different N1KV port-profile from the protected VMs. To complete the picture, the following diagram shows how the port-profiles for the protected VMs would be configured on the Nexus 1000V:

[![N1KV-Port-Profile](http://adamraffe.files.wordpress.com/2013/02/n1kv-port-profile.jpg)](http://adamraffe.files.wordpress.com/2013/02/n1kv-port-profile.jpg)

One other point worth mentioning: traffic through the ASA 1000V can only flow between the Inside and Outside interfaces; traffic cannot flow between multiple security profiles which both sit behind the Inside interface. For that, you would need to use a VM level firewall product such as Virtual Security Gateway (VSG).
