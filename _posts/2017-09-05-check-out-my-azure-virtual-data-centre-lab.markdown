---
author: Adam
comments: true
date: 2017-09-05 16:15:56+00:00
layout: post
link: http://adamraffe.com/2017/09/05/check-out-my-azure-virtual-data-centre-lab/
slug: check-out-my-azure-virtual-data-centre-lab
title: Check Out my Azure 'Virtual Data Centre' Lab!
wordpress_id: 1349
categories:
- Azure
- Security
tags:
- VDCs
---

I've just finished working on a new self-guided lab that focuses on the Azure 'Virtual Data Centre' (VDC) architecture. The basic idea behind the VDC is that it brings together a number of Azure technologies, such as hub and spoke networking, User-Defined Routes, Network Security Groups and Role Based Access Control, in order to support enterprise workloads and large scale applications in the public cloud.

The lab uses a set of Azure Resource Manager (ARM) templates to deploy the basic topology, which the user will then need to further configure in order to build the connectivity, security and more. Once the basic template build has been completed, you'll be guided through setting up the following:



	
  * Configuration of site-to-site VPN

	
  * Configuration of 3rd party Network Virtual Appliance (in this case a Cisco CSR1000V)

	
  * Configuration of User Defined Routes (UDRs) to steer traffic in the right direction

	
  * Configuration of Network Security Groups (NSGs) to lock down the environment

	
  * Azure Security Center for security monitoring

	
  * Network monitoring using Azure Network Watcher

	
  * Alerting and diagnostics using Azure Monitor

	
  * Configuration of users, groups and Role Based Access Control (RBAC)


<!-- more -->

I put this lab together to run as a workshop for the partners that I work with, but I am also making it available here for you to run if you are interested. It should take around 3 - 4 hours to complete and all you need is an Azure subscription (you can sign up for a free trial [here](https://azure.microsoft.com/en-us/free/))

The lab guide is available on Github here:

[https://github.com/Araffe/vdc-networking-lab](https://github.com/Araffe/vdc-networking-lab)

Let me know what you think!

Adam


