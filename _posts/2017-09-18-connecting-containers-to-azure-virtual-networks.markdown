---
author: Adam
comments: true
date: 2017-09-18 16:14:26+00:00
layout: post
link: http://adamraffe.com/2017/09/18/connecting-containers-to-azure-virtual-networks/
slug: connecting-containers-to-azure-virtual-networks
title: Connecting Containers to Azure Virtual Networks
wordpress_id: 1397
categories:
- Azure
- Docker
tags:
- containers
---

Azure has a number of ways in which to run containers, ranging from simple IaaS VMs running Docker, to Azure Container Service (a service that provisions a full container cluster using Kubernetes, Swarm or DC/OS) and [Azure Container Instances](http://adamraffe.com/2017/07/27/say-hello-to-azure-container-instances-aci/). One of the characteristics of these services is that when a container is provisioned, it typically has an IP address allocated to it from within the local host, rather than from the Azure virtual network to which the host is connected. As an example, consider the following scenario where we have a single Azure IaaS virtual machine running Ubuntu and Docker:

![Container-net1]({{ site.baseurl }}/img/2017/09/container-net11.jpg)<!-- more -->

In the very simple example above, we have an Azure virtual machine running Docker attached to a virtual network, which has an address range of 10.3.1.0/24. The host itself (or technically, its NIC) is allocated an IP address from the VNet range. The container itself is allocated an IP address from the Docker0 bridge address range (typically 172.17.0.0/16). Although this works fine (NAT takes place within the host to connect the container to the outside world), we lose a certain amount of visibility into the container's address space from the Azure world - so it becomes more difficult to apply Azure networking features such as Network Security Groups (NSGs). Wouldn't it be nice if we could have our containers sitting directly on an Azure VNet, with an IP address assigned from that VNet range? In fact, we can now do this using a set of network plugins, available [here](https://github.com/Azure/azure-container-networking). Let's have a look at how these plugins work.

For this example, I'll be using the CNM (Container Network Model) plugin - there is a CNI (Container Network Interface) version also available. I won't go into the differences between these two models here as it has been covered at length elsewhere (e.g. [here](http://www.nuagenetworks.net/blog/container-networking-standards/)). So the first thing I'll do is download the plugin and run it in the background (I am using the latest version, which is 0.9 at the time of writing:

{% highlight shell %}
curl -sSL https://github.com/Azure/azure-container-networking/releases/download/v0.9/azure-vnet-cnm-linux-amd64-v0.9.tar.gz
tar xzvf azure-vnet-cnm-linux-amd64-v0.9.tar.gz
sudo ./azure-vnet-plugin&
{% endhighlight %}


Now that we have the plugin running, we can create a new Docker network using the Azure driver:

{% highlight shell %}
sudo docker network create --driver=azure-vnet --ipam-driver=azure-vnet --subnet=10.3.1.0/24 azure e7b58e62fb381c0e5429b2d2b38520eb02513b2e6812d0c1a00a2ba7691bfeda
{% endhighlight %}


Let's break down the above command: first, note that we are creating a network called 'azure' using the azure-vnet driver, but also that we are using the azure-vnet IPAM driver for IP address management. This IPAM plugin is necessary for allocating IP addresses to our containers directly from the Azure fabric. For the subnet, we need to match this with the subnet within the Azure VNet that we are connecting to. Let's verify what we just created:

{% highlight shell %}
adraffe@Docker-Host:~$ sudo docker network ls
NETWORK ID NAME DRIVER SCOPE
e7b58e62fb38 azure azure-vnet local
9894d83e1ebc bridge bridge local
3697ca264bfa host host local
64eb1143821e none null local
{% endhighlight %}


Now that we have the network created, let's have a look at what has been created on the host using ifconfig:

{% highlight shell %}
adraffe@Docker-Host:~$ ifconfig
azure2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 10.3.1.4 netmask 255.255.255.0 broadcast 0.0.0.0
inet6 fe80::20d:3aff:fe29:f117 prefixlen 64 scopeid 0x20<link>
ether 00:0d:3a:29:f1:17 txqueuelen 1000 (Ethernet)
RX packets 248 bytes 125708 (125.7 KB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 356 bytes 60542 (60.5 KB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

{% endhighlight %}

Here, we can see that a new bridge has been created called 'azure2'. This sits alongside the standard 'Docker0' bridge and will be used for connecting our containers to once they have been created. Note that the IP address of this bridge is 10.3.1.4 - this address happens to be the IP of the Docker host itself, allocated from the Azure virtual network.

OK, so let's create a container and try to connect it to our VNet. I'll spin up a basic Alpine image and connect it to the network created above:

{% highlight shell %}
adraffe@Docker-Host:~$ sudo docker run -it --net=azure alpine
docker: Error response from daemon: No address returned.
{% endhighlight %}


Hmm, that didn't work - it seems that I'm not getting an IP address allocated to my container. Why is this? The reason is that - right now - we need to _pre-allocate_ IP addresses in Azure in order to make them available to containers. This could of course change (and hopefully will) in the future. In order to pre-allocate an address, I need to create an additional IP config and apply it to my Docker host's NIC. I could do this in a number of ways (portal, ARM templates, etc), but I'll use the Azure CLI here:

{% highlight shell %}
az network nic ip-config create -g Docker --nic-name docker-host191 --private-ip-address 10.3.1.5 --name IPConfig2
{% endhighlight %}


Now that we have this additional IP address in place, let's try creating the container again:

{% highlight shell %}
adraffe@Docker-Host:~$ sudo docker run -it --net=azure alpine
/ # ifconfig
eth0 Link encap:Ethernet HWaddr B6:EC:0A:16:CF:59
inet addr:10.3.1.5 Bcast:0.0.0.0 Mask:255.255.255.0
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:7 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:586 (586.0 B) TX bytes:0 (0.0 B)

{% endhighlight %}


This time, the container is created - doing an ifconfig from within the container shows that it has an IP address of 10.3.1.5, which sits directly on the Azure VNet I am using. Let's have a look at what this looks like:

![Container-net2]({{ site.baseurl }}/img/2017/09/container-net2.jpg)


### **Connecting Kubernetes Clusters to Azure Virtual Networks**


The example above was pretty simple - just a single host running Docker with a very basic container setup. What if I want a full Kubernetes cluster connected in to my Azure VNet?

By default, ACS with Kubernetes uses a basic network plugin called kubenet. With this approach, Kubernetes pods are deployed to a subnet that is different to those residing in the Azure VNet. The [ACS Kubernetes plugin](https://github.com/Azure/azure-container-networking/blob/master/docs/acs.md) works in a very similar way to the example I showed in the first section above - a number of additional IP addresses are added to the host's NICs and are then allocated to Kubernetes pods as they are created, allowing pods to sit directly on Azure VNets and allowing full use of the Azure SDN features, such as Network Security Groups.

Thanks for reading!


