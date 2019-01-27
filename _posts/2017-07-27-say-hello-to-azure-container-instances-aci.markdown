---
author: Adam
comments: true
date: 2017-07-27 08:42:10+00:00
layout: post
link: http://adamraffe.com/2017/07/27/say-hello-to-azure-container-instances-aci/
slug: say-hello-to-azure-container-instances-aci
title: Say Hello to Azure Container Instances (ACI!)
wordpress_id: 1226
categories:
- Azure
tags:
- containers
---

It's been some time since I last posted a blog - I've been spending the majority of my time settling in to my new role at Microsoft and learning as much as I can about the Azure platform. Considering my background, it seems fitting that my first "Azure related" blog post is all about....ACI! In this case however, ACI stands for **A**zure **C**ontainer **I**nstances - a new Azure offering announced this week and currently in preview. So what are Azure Container Instances?<!-- more -->

Let's first consider how we might run containers in the cloud today. Up until this point, if you wanted to run containers on Azure, you had two main options:

a) Provision your own virtual machines and run containers on top, or

b) Use Azure Container Service (ACS) - this service essentially allows you to quickly provision a cluster using either Kubernetes, DC/OS or Docker Swarm as the orchestration engine.

Both of these solutions suffer (to varying degrees) from the overheard of having to manage the underlying VM infrastructure and everything that goes along with that - maintenance, security, patching and so on. Wouldn't it be nice if we could simply create our containers in the cloud without having to worry about the infrastructure they are running on? Look no further than Azure Container Instances.

ACI instances give us the ability to run containers easily from the command line or using ARM templates, with no need to create the underlying VMs. In addition, these container instances are billed _per second, _meaning they should work out to be extremely cost-effective. The official MS page for ACI is [here](https://azure.microsoft.com/en-us/services/container-instances/).

To see how ACI works, let's try it out. To start with, I'm going to create a single container using the Azure CLI. I'll use the following command:

    
    az container create -g ACI --name acitest --image nginx --ip-address public


This command is essentially asking for a container to be created under the resource group named "ACI", with a name of "acitest", using the public nginx image (which will be pulled from Docker Hub). The command also asks for a public IP address to be assigned to the container. The results of this can be seen in the following figure:

![ACI1](https://adamraffe.files.wordpress.com/2017/07/aci11.jpg)



The JSON output that results shows a wealth of information about the container - for example, I can see the amount of CPU and memory (1 CPU core, 1.5Gb of RAM - this is configurable), the IP address that has been assigned as well as some information about the resource group. You can also see that the provisioning state is "creating". If I run the command _az container list _a few seconds later, I can see the container state is now "succeeded":

![ACI2](https://adamraffe.files.wordpress.com/2017/07/aci2.jpg)

At this point, I can browse to the public IP address that Azure assigned to my container and I receive the default nginx page.

I can also get the log output from the container using the command _az container logs_:

![ACI4](https://adamraffe.files.wordpress.com/2017/07/aci4.jpg)

OK, so this is all very nice - but wouldn't it be great if we could automate this process? And it would be even better if we could create multiple container instances at the same time. It is of course possible to do this using ARM templates. Check out the example template below (full template available [here](https://github.com/Araffe/armtemplates/tree/master/containers/aci-demo)):

![ACI5](https://adamraffe.files.wordpress.com/2017/07/aci51.jpg)

The JSON shown above should do the following:



	
  * Create a new container group named "aciMultiGroup".

	
  * Create two container instances - "aci-container1" and "aci-container2" as part of the group.

	
  * Expose port 80 from container 1 and assign a public IP address to the group.


This template introduces us to the container group concept. The idea behind this is that one or more containers with similar requirements or lifecycles can be deployed and managed together inside a container group (somewhat similar to the 'pod' concept in Kubernetes).

So let's deploy this template and see what happens:

![ACI6](https://adamraffe.files.wordpress.com/2017/07/aci6.jpg)

OK, everything looks good. The output of the script has given me the public IP address of the container group - let's try and browse to that:

![ACI7](https://adamraffe.files.wordpress.com/2017/07/aci7.jpg)

Success! Hopefully this post has given you a taster of what Azure Container Instances can do and the potential they have. Thanks for reading!
