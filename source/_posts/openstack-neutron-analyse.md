title: OpenStack Neutron解析
date: 2014-05-22 19:08:24
categories: [OpenStack]
tags: [OpenStack,Neutron]
---

![](http://ww2.sinaimg.cn/large/7458d655gw1f8mfc0qf7cj20g405kt9d.jpg)

> 很久之前写了一篇关于OpenStack Neutron解析的文章，那时只是粗略的写了一下把Neutorn的整体架构分析了一下，后来一直忙于其他事情，也就忘了去详细分析一下Neutron的架构。这次这篇算是完成未完之事，同时也是对之前的一个知识的总结及恢复。

<!--more-->

OpenStack的Neutron自从由nova-network从Nova中分离出来之后，一直感觉十分的不稳定，而且初期其结构也是十分的复杂。很多人刚刚接触Neutron，甚至刚刚接触OpenStack的时候，都是被困在Neutron异常复杂的机制。尤其当我们部署了一套由Neutron管理网络的OpenStack环境时，会发现很多时候都是在解决各种莫名奇妙的问题，但我们纠察问题时，总是会涉及Neutron。所以，我总是一直认为在实际的生产环境中，如果不是对于网络真的有着很特殊的需求，直接部署OpenStack的Essex版本，别人问我，我也是如是的回答。但是，我们如果是想研究OpenStack的话，Neturon的可玩性还是很大的，尤其是其支持SDN等一些很前瞻性的特性。所以，对于Neturon我们有必要深入的研究一番。

接着上次的那篇[文章](http://panpei.net.cn/2013/12/04/openstack-neutron-mechanism-introduce/)，我们再来重新回顾一下Neutron的架构，从物理上划分的话，我们的Neutron主要部署在两类节点上：Compute节点和Network节点，而至于Controller节点，那不是主要的所在，因为几乎所有的组件都要在部署一个server服务在Controller节点上。从网络分层上来看，主要分为二层网络L2-Agent，三层网络L3-Agent，以及DHCP-Agent。借用官网上几张图片，按照物理划分的方式，大致分析一下Neutron架构。

## Compute节点
![](/img/2014/05/22/under-the-hood-scenario-1-linuxbridge-compute.png)

这张图摘自[官网](http://docs.openstack.org/admin-guide-cloud/content/under_the_hood_openvswitch.html#under_the_hood_openvswitch_scenario1)，为Compute节点的网络架构及流程分析图。这张图中，我们可以清楚的看到网络相关的一些设备（其实就是一些进程或者系统接口）被分为四类：`TAP device`，`veth pair`，`Linux Bridge`，`Open vSwitch`，这里其实用的Open vSwitch的方式部署Neutron，而我之前也一直使用Open vSwitch部署的，但是这里却也是有着Linux Bridge的，正如之前[文章](http://panpei.net.cn/2013/12/04/openstack-neutron-mechanism-introduce/)所言，这个是为了实现安全组功能，但Open vSwitch暂不支持OpenStack的实现方式，所以只好用Linux Bridge实现qbr网桥作为一个折衷方案。接下来就是最为大头的Open vSwitch，它在Neutron中构建了一个虚拟的交换机，而这个虚拟交换机由L2-Agent控制着，最终所有Compute节点中的虚拟交换机统一构成一个巨大的虚拟交换机，统一控制虚拟机在二层网络的数据交换和接入功能。从图中我们可以看出每个Compute节点有两个Open vSwitch网桥，其实`br-int`才是真正扮演交换机角色的，而`br-eth1`则是通过`GRE`通道在所有节点之间构成一个统一的通信层，实现各个Compute节点上虚拟机之间的通信。

## Network节点
接下来就是看看Network节点上的情况，下面的是Network节点的原理图：

![](/img/2014/05/22/under-the-hood-scenario-2-ovs-network.png)

这张图上，我们可以清楚看到Network节点被分为三大部分：`Configured by L2-Agent`，`Configured by L3-Agent`，`Configured by DHCP-Agent`，而L2-Agent那部分和我们在Compute节点讨论的是一致的，所以此处就忽略了。接着我们看L3-Agent，在Neutron中，出现了私有网络这一概念，当然也是实际存在的。而依据以前的nova-network，是无法实现这一功能的，nova-network顶多能使用VLAN技术实现网络隔离，而无法实现真正的私有网络。在物理网络中，我们要想实现一个私有网络，那么就必须有个路由器才行，而L3-Agent正是Neutron中实现这个路由器而存在的（事实上，我们在Havana版本中部署的环境中，网络拓扑图中就很形象的显示了这些routers），L3-Agent的底层实现采用的是Linux系统自带的iptables技术，通过动态的生成配置iptables规则，实现网络的路由功能以及floatip功能。而每个用户都要一个私有网络的话，一个路由肯定是不够用的，接着就是Linux中namespace技术的用武之地了，事实上，Neutron为每个私有网络都配置一个router和dnsmasq，而这个就是依靠namespace进行规则和配置的隔离。如下图所示：

![](/img/2014/05/22/under-the-hood-scenario-2-ovs-netns.png)

现在，每个私有网络可以建立了，但是我们总不能为每个虚拟机手动配置私有ip的，所以这时候DHCP-Agent就显示了其用处，DHCP-Agent通过控制dnsmasq实现了DHCP功能，这样每个虚拟机在启动的时候就可以动态获取私有ip了。

## 总结
至此，我们就已经把OpenStack Neutron的整理结构分析清楚了，剩下就是我的上篇[文章](http://panpei.net.cn/2013/12/04/openstack-neutron-mechanism-introduce/)中的流程打通了。

## 参考
1. [Networking in too much detail](http://openstack.redhat.com/Networking_in_too_much_detail)
2. [OpenStack Cloud Administrator Guide](http://docs.openstack.org/admin-guide-cloud/content/under_the_hood_openvswitch.html#under_the_hood_openvswitch_scenario1)
3. [OpenStack 网络：Neutron 初探](http://www.ibm.com/developerworks/cn/cloud/library/1402_chenhy_openstacknetwork/)
4. [Neutron中的iptables](http://blog.csdn.net/lynn_kong/article/details/13503847)
