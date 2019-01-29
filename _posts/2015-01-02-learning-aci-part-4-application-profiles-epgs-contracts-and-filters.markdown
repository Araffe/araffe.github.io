---
author: Adam
comments: true
date: 2015-01-02 17:08:11+00:00
layout: post
link: http://adamraffe.com/2015/01/02/learning-aci-part-4-application-profiles-epgs-contracts-and-filters/
slug: learning-aci-part-4-application-profiles-epgs-contracts-and-filters
title: 'Learning ACI - Part 4: Application Profiles, EPGs, Contracts and Filters'
wordpress_id: 778
categories:
- ACI
- Nexus 9000
tags:
- APIC
---

In this post, we'll take a closer look at some of the most important constructs within the ACI solution - application profiles, End Point Groups (EPGs), contracts and filters. Hopefully you've taken a look at the other parts in this series - in [part 1](http://adamraffe.com/2014/12/03/learning-aci-part-1-overview/), I gave a brief overview of ACI and what I would be covering in the series. [Part 2](http://adamraffe.com/2014/12/03/learning-aci-part-2-bringing-up-a-fabric/) discussed the fabric bring-up process, with [part 3](http://adamraffe.com/2014/12/03/learning-aci-part-3-getting-familiar-with-the-apic/) giving a short tour of the APIC.<!-- more -->

**End Point Groups**

You'll hopefully be aware by now that ACI introduces a number of new concepts that provide new ways of defining connectivity using a policy based approach. Of these, arguably the most important is the _End Point Group (EPG). _What is an EPG and how is it used within ACI? Well, the basic idea is that you define groups of servers, VMs, IP storage or other devices that have common policy requirements - once these devices are grouped together, it becomes much easier to define and apply policy to the group, rather than to individual end points.

Let's imagine you have a standard three-tiered type of application (I'm aware that this is a rather overused example, but it serves its purpose in this case):

[![3-tiered-app]({{ site.baseurl }}/img/2015/01/3-tiered-app.png)]({{ site.baseurl }}/img/2015/01/3-tiered-app.png)

Within each of our application tiers, we have a number of 'end points' - we don't really care too much what those are (bare metal, VMs, etc). The important thing is that these end points all require the same policy to be applied. If we can identify which group these end points need to be part of, we can create the corresponding EPGs within ACI:

[![3-tiered-epg]({{ site.baseurl }}/img/2015/01/3-tiered-epg.png)]({{ site.baseurl }}/img/2015/01/3-tiered-epg.png)

How do we define which end points reside in which EPG? That can be done either statically or dynamically and depends somewhat on whether we are talking about bare metal hosts or virtual machines. If these are VMs, then ACI can integrate closely with the virtual machine manager (such as vCenter), with the process of attaching a VM to a network resulting in that VM becoming part of the desired EPG. Now that we have our EPGs defined and end points residing within them, what comes next? At this point, there are two important concepts to understand:

1) Within an EPG, communication is free-flowing - an end point can communicate freely with another end point within the same EPG, regardless of where those EPs sit physically or virtually.

2) Between EPGs, _no communication is permitted by default - _if you do nothing else at this point, an EP residing in the "Web" EPG will not be able to communicate with an EP in the "App" EPG.

Now that we know these rules, it follows that we need to put something extra in place if we want inter-EPG communication. This is where contracts and filters come in.

**Contracts and Filters**

What does a contract do? Essentially, it is a policy construct defining the potential communication between the EPGs on the system. A contract can be very restrictive (for example, allowing only one port), or it can be completely open ("permit any") depending on your requirements.

[![Contracts]({{ site.baseurl }}/img/2015/01/contracts.png)]({{ site.baseurl }}/img/2015/01/contracts.png)

A contract will refer to one or more _filters_. The filter is the construct used to actually define the specific protocols and ports required between EPGs. In the following example, I am creating a filter with a single entry - HTTP:

[![Filter-example]({{ site.baseurl }}/img/2015/01/filter-example.png)]({{ site.baseurl }}/img/2015/01/filter-example.png)

This filter can now be referenced when creating a contract - in this case, the filter is a 'subject' that gets defined under the contract:

[![Contract-Subject]({{ site.baseurl }}/img/2015/01/contract-subject.png)]({{ site.baseurl }}/img/2015/01/contract-subject.png)

Note that you can apply the contract to both directions if bi-directional communication is required between EPGs.

**Application Profiles
**

OK, so we have some EPGs defined and the contracts / filters we need to communicate between them. We now need some way of tying all of this together - an _Application Profile_ is simply a collection of EPGs and the policies needed to communicate between them:

[![App-Profile]({{ site.baseurl }}/img/2015/01/app-profile.png)]({{ site.baseurl }}/img/2015/01/app-profile.png)

During the process of creating an application profile and its associated EPGs, you will associate one or more contracts with the EPGs that you create. When a contract is associated with an EPG, that EPG can either _provide_ or _consume_ the contract. Let's imagine our "Web" EPG is providing a service using HTTP (which seems perfectly reasonable), and that this service needs to be accessed from the "App" EPG. In that case, the "Web" EPG would _provide_ a contract (and its associated filter) allowing HTTP. The "App" EPG would _consume_ the very same contract. The APIC actually gives us a nice graphical representation of this relationship:

[![Provider-Consume]({{ site.baseurl }}/img/2015/01/provider-consume.png)]({{ site.baseurl }}/img/2015/01/provider-consume.png)

**Contract Scope**

When defining a contract, you can specify the _scope_ of that contract - for example, Private Network, Application Profile, Tenant or Global. The scope of a contract defines the level of policy enforcement between the various entities within ACI. As an example, let's say we create a single contract called "web-to-db". Now let's say we create two application profiles (app1 and app2), with the EPGs within those app profiles providing and consuming our single contract, as shown here:

[![Contract-Scope]({{ site.baseurl }}/img/2015/01/contract-scope.png)]({{ site.baseurl }}/img/2015/01/contract-scope.png)

If our contract scope is set to "application profile", this means that communication is _not_ permitted between application profiles (for example, no communication between 'App1-Web' and 'App2-Web'). If we were to set the scope of our contract to 'Private Network', then communication would be possible between all four EPGs (assuming everything is part of one private network).

That's it for this instalment - next up, we'll look at some of the networking elements within ACI, such as bridge domains and private networks (contexts). Part 5 is [here](http://adamraffe.com/2015/01/06/learning-aci-part-5-private-networks-bridge-domains-and-subnets/).
