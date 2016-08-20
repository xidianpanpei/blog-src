title: OpenStack Guide:Create Your First VM With Horizon
date: 2013-11-30 16:31
categories: [OpenStack]
tags: [OpenStack,Horizon,VM]
---

Recently I am doing some work about OpenStack, and find that OpenStack is very complex and the materials on the Internet can afford enough information to us during learning OpenStack.

I have found a good guide which teach us to build the virtual network and create the VMs, this guide is written by Yongsheng Gong who is one of the core developers of Neutron. While this guide is written with Chinese, maybe some others exclude Chinese will unstand it hard, so I decide to translate it to English version.

Source From: [The Guide of Neutron Network][http://www.ustack.com/blog/neutron_intro/]

## Handle the network on Horizon
After having planned our network, we can do some operation on Horizon.

### Create the external network with the admin authority
We hava said that the external network needs to be created by admin. If the ip address which we get is 20.0.2.0/24, and we use the ips from 20.0.2.10 to 20.0.2.20, the gateway is 20.0.2.1. Then we can create a external network and a subnetwork by the admin authority:

#### 1.First, we should login in the system as admin, then choose the Admin Panel, and the panel will show the current network list after clicking 'Networks':

![]()