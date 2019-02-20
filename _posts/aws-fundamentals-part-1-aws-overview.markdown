---
author: Adam
comments: true
date: 2019-02-20 21:00:12+00:00
layout: post
slug: aws-fundamentals-part-1-aws-overview
title: 'AWS Fundamentals - Part 1: AWS Overview'
---

In August 2018, I joined Amazon Web Services as a Solution Architect. Although I was previously at Microsoft working on Azure, the two platforms are somewhat different, so I had to spend a large portion of my time getting up to speed with AWS. There are a huge amount of training courses, white papers and blog posts devoted to all aspects of the AWS platform, however one thing I noticed was that it was difficult to find a good 'fundamentals' type guide in one place. This blog post series is my attempt at addressing this gap. I started writing these posts primarily because writing things down helps to cement the knowledge in my mind - but if it helps others, then great.

One thing to point out is that nothing new is being provided here - all the info I'm providing in these posts is available elsewhere (AWS documentation, various training courses, etc) - the point is that writing these posts helps me to learn and is how I would like to see the information flow if I were reading it for the first time.

So, first things first - let's look at what AWS actually _is_.


# What is AWS?


AWS is a cloud computing platform. At the most basic level, cloud platforms such as AWS provide an alternative to on-premises data centres and infrastructure. The cloud vendor is ultimately responsible for the underlying infrastructure that a customer would otherwise have to manage - facilities, hardware, power, cooling and so on. The cloud vendor typically provides a wide range of services that customers can make use of - these services range from 'basic' infrastructure such as compute, network and storage all the way up to machine learning and AI based offerings.

Cloud environments such as AWS typically provide a number of ways to interact with the platform - new users can easily provision services (e.g. creating a virtual machine) through an online portal or via a command line interface. More advanced users may choose to deploy services in a more automated fashion - using 'infrastructure as code' type tools such as AWS CloudFormation or Terraform from Hashicorp, or using an API driven approach.

As an example, an AWS user can request a virtual machine (known as an _instance_ in AWS terminology) with a specific amount of CPU and memory and with the operating system of their choice. That user can then choose a particular storage configuration (e.g. number and size of attached disks) and can also select the networking configuration of their choice (for example, number of network interfaces, whether the VM has access to the Internet, etc). Once the user has chosen their configuration, they can deploy the VM within a few minutes into the location of their choice (see _regions_ section below).

One of the key points about cloud environments is that users pay _only for what they consume_. In the example above, our user deploys their VM, makes use of it for their specific purpose and then has the option to tear down the VM once they are finished with it. The user will only need to pay the cloud provider for the time that VM has actually been up and running.


# AWS Regions


Thinking about our example in the previous section, if a user deploys a virtual machine using AWS, where does that VM actually "go"? Does the user have any control at all over which location his or her VM ends up in? How do we even know which country our resources will reside in?

To answer this question, AWS offers its services from data centres it has deployed all over the world - these data centres are deployed into different geographic areas known as _regions_. When a user provisions a service in AWS, they have the choice to deploy that service into a specific region. AWS is rolling out more regions over time - it's kind of pointless me giving the figures on this post as they are likely to be out of date within a few months, so you can find the up to date AWS regions list [here](https://aws.amazon.com/about-aws/global-infrastructure/).

One of the key characteristics of a region is that it provides a very high degree of isolation from other regions. The idea is that, should something happen within a particular region that takes it offline, regions in other locations will be unaffected.


# Availability Zones


A region is further subdivided into a number of _Availability Zones_. An AZ also provides a high level of isolation - the difference here is that AZs residing within a region are connected together using extremely low latency links.

To understand how AZs should be used, here's an example. Let's say you have a service you want to deploy consisting of a number of redundant virtual machines (say 3). It's quite likely that you _don't_ want those VMs to all be deployed into the same physical facility and would prefer them to be distributed across multiple facilities. In that case, you have the option (and in fact it would be recommended) to deploy each of your 3 VMs into different AZs.

Here's an example. In the diagram below, we have two regions where we are deploying services (Ireland and Oregon). Each of those regions has three Availability Zones (AZs) and we are deploying some services across these AZs (EC2 instances and database instances).

![Regions-AZs]({{ site.baseurl }}/img/2019/Regions-AZs.jpg)


# AWS Services


OK, when looking at the services AWS offers for the first time, the most important thing is not to freak out, because the AWS platform is _absolutely enormous._ When I first started to look at what AWS offered, I was blown away by just big and comprehensive this platform really is. For evidence of this, take a quick look at the [AWS Products](https://aws.amazon.com/products/) web page - you'll see here that there are hundreds of services and products available ranging from compute, storage and networking, through databases, security services and containers, all the way up to machine learning and Internet of Things (IOT) offerings. Not only that, but there are new services and updates happening _constantly_. In short, it's pretty much impossible to be an expert in everything that AWS offers, no matter how hard you might try.

Let's look at some of the services that AWS offer in some more detail (this is not an exhaustive list by any means!):

## Compute:

	
  * [EC2](https://aws.amazon.com/ec2/) - Provides the ability to run virtual machines in the cloud.

	
  * [Lambda](https://aws.amazon.com/lambda/) - Amazon's 'serverless' compute offering, allows users to run code in response to events, without having to provision or manage servers.

	
  * [ECS](https://aws.amazon.com/ecs/), [EKS](https://aws.amazon.com/eks/) and [Fargate](https://aws.amazon.com/fargate/) - these form the backbone of the AWS container offerings. ECS provides managed container clusters, while Fargate provides a form of 'serverless' containers (i.e. the ability to run containers without having to manage the underlying instances). EKS provides a managed Kubernetes service for container orchestration.

	
  * [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) - PaaS service providing an application hosting environment, giving the ability to deploy apps without worrying about underlying infrastructure.


## Storage:

	
  * [S3 and Glacier](https://aws.amazon.com/s3/) - Provides object based storage and archival capabilities in the cloud.

	
  * [EFS](https://aws.amazon.com/efs/) - Elastic File System. Provides scalable file services for use with other cloud and on-premises resources.

	
  * [Snowball](https://aws.amazon.com/snowball/) / [Snowmobile](https://aws.amazon.com/snowmobile/) - solutions for transporting large amounts of data into the cloud.


## Networking:

	
  * [Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/) - a logically isolated network inside the AWS cloud, with complete control over subnetting, access controls, etc.

	
  * [Direct Connect](https://aws.amazon.com/directconnect/?p=tile) - dedicated connectivity between on-premises DCs and the AWS cloud.

	
  * [Elastic Load Balancer](https://aws.amazon.com/elasticloadbalancing/) - distributes traffic across multiple targets (VMs, containers, etc). Comes in two main versions - Network Load Balancer (layer 4) and Application Load Balancer (layer 7).


This list barely scratches the surface of what's available in AWS - I certainly won't be covering every service in this blog series, but instead will focus on the services I think are necessary to consider yourself well versed in the 'fundamentals' of AWS.

Finally, let's look at where to go for more information about AWS.


# AWS Documentation and Samples


The main AWS documentation is at [https://aws.amazon.com/documentation/](https://aws.amazon.com/documentation/). This is a good starting point if you want to learn about any of the available services in AWS. From each of the product areas, you'll find user guides, case studies, white papers and more.

You can also find a huge collection of sample applications, workshops at [https://github.com/aws-samples](https://github.com/aws-samples).

That's about it for the first post. In the next post we'll have a look at setting up an AWS account, as well as monitoring important items such as billing.
