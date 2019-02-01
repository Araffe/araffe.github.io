---
author: Adam
comments: true
date: 2014-12-03 16:50:07+00:00
layout: post
link: http://araffe.github.io/2014/12/03/learning-aci-part-1-overview/
slug: learning-aci-part-1-overview
title: 'Learning ACI - Part 1: Overview'
wordpress_id: 702
categories:
- ACI
- Nexus 9000
tags:
- APIC
---

This post is the first in a series in which I'm going to describe various aspects of Cisco's Application Centric Infrastructure (ACI). ACI, if you aren't already aware, is a new DC network architecture from Cisco which uses a policy based approach to abstract traditional network constructs (e.g. VLANs, VRFs, IP subnets, and many more). What I'm _not_ going to do in these posts is cover too many of the basic concepts of ACI - for that, I recommend you read the _ACI Fundamentals _book on cisco.com, available [here. ](http://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/aci-fundamentals/b_ACI-Fundamentals.html)Instead, my intention is to cover the practical aspects of building and running an ACI fabric, including how to bring up a fabric, basic physical connectivity and integration with virtualisation systems.<!-- more -->

Before we start, it's worth taking a look at the high level architecture of ACI. Here are the highlights:



	
  * ACI is based primarily upon two components: Nexus 9000 switches and Application Policy Infrastructure Controllers (APICs). The Nexus 9000 platform forms the physical switching infrastructure, while the APIC is a clustered policy management system responsible for all aspects of fabric configuration.

	
  * ACI uses a _leaf and spine_ topology. A given leaf node is connected to all spine nodes in the fabric, with no connectivity between leaves or between spine switches. All server, host, services and external connectivity is via leaf nodes - nothing is directly connected to a spine (apart from leaf nodes, obviously).

	
  * ACI uses a _policy model_ to define how applications and attached systems communicate. In addition, policies are used within ACI to define almost every aspect of system configuration and administration.

	
  * Several new constructs are introduced within ACI, including _End Point Groups, Application Profiles, Contracts, Filters,_ as well as objects associated with external connectivity, such as _Layer 2 and Layer 3 Outsides_. I'll be covering most of these in the forthcoming posts, but please have a look through the relevant sections of the ACI fundamentals book (linked to above) for more details.

	
  * ACI has the ability to integrate closely with L4 - L7 services devices using the concept of _service graphs_. A service graph is essentially a description of where a given service (e.g. a firewall) should sit in the flow of traffic. Configuration of the services device can also move directly to the APIC through the use of device packages.

	
  * All ACI fabric functionality is exposed through a Northbound REST API, with both XML and JSON supported.


Now that we've covered some of the basics, I'll run quickly through what I plan for the rest of this blog series:

	
  1.  First things first: getting a fabric up and running. Bringing up a fabric for the first time is a relatively easy thing to do but I'll run through how it's done and some of the key information you'll need to build a 'bare' fabric.

	
  2. Once we have the fabric up and running, we'll take a look at the APIC and familiarise ourselves with the overall look and feel, as well as running through the main sections (tenants, fabric, and so on).

	
  3. We'll then take a look at some of the major objects within ACI: Application Profiles, EPGs, Contracts and Filters. It's important that these concepts are well understood, so I'll examine these in more detail.

	
  4. Next on the list: networking concepts such as contexts and bridge domains - and how they relate to each other.

	
  5. Before we look at getting some real connectivity between hosts, we need to take a look at _access policies_ - we use these to define physical connectivity into the fabric.

	
  6. I'll then bring everything together and show you how we get two hosts talking to each other through the fabric.

	
  7. Further down the line, we'll take a look at some other topics including external connectivity, integration with L4-L7 services and more!


OK, that's enough talking - let's get going! [Part 2 is here.](https://araffe.github.io/aci/nexus%209000/2014/12/03/learning-aci-part-2-bringing-up-a-fabric)


