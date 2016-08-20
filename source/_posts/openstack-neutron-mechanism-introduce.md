title: OpenStack Neutron运行机制解析概要
date: 2013-12-04 18:40
categories: [OpenStack]
tags: [OpenStack,Neutron]
---

自从开学以来，玩`OpenStack`也已经3个月了，这段时间主要把精力投在了`OpenStack`的安装部署和网络组件`Neutron`的研究上了。这期间零零散散在安装部署和`Neutron`运作原理上来回切换，有点在实践中学习的味道，虽然在安装部署的过程遇到了不少的问题，也一一都给解决了。然而，总是觉得对于`Neutron`的机制理解的还是不够透彻。前一阵子刚刚部署好一套`Havana`版本的系统，并开始投入使用。这阵子又开始投入`OpenStack`的IPv6的试验中，因为需要对底层的路由机制进行彻底的了解，所以不得不又开始重新捋了一遍这方面的知识，在网上Google了一把资料，这次算是彻底的把`Neutron`的运作机制彻底的理清楚了，之前那些半懂不懂的东西，现在也是觉得豁然开朗了。

<!--more-->

这次在[RDO][1]找到了想要的资料，至少这篇资料对于之前的那些零散的知识做了一个很好的连贯，对于自己而言，算是把`Neutron`讲解彻底明了,现在把那篇材料翻译下再糅合点自己的理解吧。

借用原资料上的一张架构图为开篇：

![](/img/2013/12/04/Neutron_architecture.png)

从这张架构图中，我们可以明显的看到有两个物理主机，计算节点和网络节点，这是因为采用了网络节点集中式的部署方式。至于为什么采用这种部署方式，那是因为自从E版之后，`OpenStack`开始把`network`功能从`Nova`中分离出来，使之成为独立的`Neutron`组件。而坑爹的是，分离后的版本，反而不支持网络分布式部署的特性了，所以目前的`Grizzly`和`Havana`版本都是只能使用网络集中式部署方案的，或者说，集群中只能存在一个部署网络功能的节点。

## 计算节点：虚拟机实例的网络
从图中看，这段网络包含了A、B、C这三段的流程。A就是虚拟机`test0`的虚拟网卡，这块没什么好讲的。和它相连的B倒是值得好好讲一下。B是一个`TAP`设备，通常是以`tap`开头的一段名称，它挂载在`Linux Bridge qbr`上面。那什么又是`TAP`设备呢？[Linux 中的虚拟网络][2]中给出了这样的解释：

>TAP是一个虚拟网络内核驱动，该驱动实现Ethernet设备，并在Ethernet框架级别操作。TAP驱动提供了Ethernet "tap"，访问Ethernet框架能够通过它进行通信。

总而言之，`TAP`设备其实就是一个Linux内核虚拟化出来的一个网络接口。OK，我们明白了`TAP`设备了，如果还是不明白可以查看`TAP`的具体[定义][3]。接下来就是`qbr`，这之前就说了，是一个`Linux Bridge`，很是奇怪，我们在这个架构中，使用的`OpenvSwitch`实现虚拟交换设备的，为什么会出现`Linux Bridge`呢？[OpenStack Networking Administration Guide][4]给出了这样的解释：

>Ideally, the TAP device vnet0 would be connected directly to the integration bridge, br-int. Unfortunately, this isn't possible because of how OpenStack security groups are currently implemented. OpenStack uses iptables rules on the TAP devices such as vnet0 to implement security groups, and Open vSwitch is not compatible with iptables rules that are applied directly on TAP devices that are connected to an Open vSwitch port. 

其实就是说，`OpenvSwitch`不支持现在的`OpenStack`的实现方式，因为`OpenStack`是把`iptables`规则丢在`TAP`设备中，以此实现了安全组功能。没办法，所以用了一个折衷的方式，在中间加一层，用`Linux Bridge`来实现吧，这样，就莫名其妙了多了一个`qbr`网桥。在`qbr`上面还存在另一个设备C，这也是一个`TAP`设备。C通常以`qvb`开头，C和`br-int`上的D连接在一起，形成一个连接通道，使得`qbr`和`br-int`之间顺畅通信。

## 计算节点：集成网桥(br-int)的网络
刚才说到D(这也是一个`TAP`设备)在br-int上面，现在轮到`br-int`出场了，`br-int`是由`OpenvSwitch`虚拟化出来的网桥，但事实上它已经充当了一个虚拟交换机的功能了。`br-int`的主要职责就是把它所在的计算节点上的VM都连接到它这个虚拟交换机上面，然后利用下面要介绍的`br-tun`的穿透功能，实现了不同计算节点上的VM连接在同一个逻辑上的虚拟交换机上面的功能。这个似乎有点拗口，其实就是在管理员看来，所有的VM都是连接在一个虚拟交换机上面的，但事实上这些VM又都是分布在不同的物理主机上面。OK，回到D上面，D通常以qvo开头。在上面还有另一个端口E，它总是以`patch-tun`的形式出现，从字面就可以看出它是用来连接`br-tun`的。

## 计算节点：通道网桥(br-tun)的网络
`br-tun`在上面已经提及了，这同样是`OpenvSwitch`虚拟化出来的网桥，但是它不是用来充当虚拟交换机的，它的存在只是用来充当一个通道层，通过它上面的设备G与其他物理机上的`br-tun`通信，构成一个统一的通信层。这样的话，网络节点和计算节点、计算节点和计算节点这样就会点对点的形成一个以`GRE`为基础的通信网络，互相之间通过这个网络进行大量的数据交换。这样，网络节点和计算节点之间的通信就此打通了。而图中的G、H正是描画这一通信。

## 网络节点：通道网桥（br-tun）的网络
正如前面所说，网络节点上也是存在一个`br-tun`，它的作用同计算节点上的`br-tun`如出一辙，都是为了在整个系统中构建一个统一的通信层而存在的。所以，这一部分的网络同计算节点上的网络的功能是一致的，因此，也就没有必要多说了。

## 网络节点：集成网桥（br-int）的网络
网络节点上的`br-int`也是起了交换机的作用，它通过I、J与`br-tun`连接在一起。最终的要的是，在这个虚拟交换机上，还有其他两个重要的`tap`设备M、O，它们分别同N、P相连，而N、P作为`tap`设备则是分别归属两个`namespace`router和dhcp，没错，正如这两个`namespace`的名称所示，它们承担的就是router和dhcp的功能。这个router是由`l3-agent`根据网络管理的需要而创建的，然后，该router就与特定一个子网绑定到一起，管理这个子网的路由功能。router实现路由功能，则是依靠在该`namespace`中的`iptables`实现的。dhcp则也是`l3-agent`根据需要针对特定的子网创建的，在这个`namespace`中，`l3-agent`会启动一个`dnsmasq`的进程，由它来实际掌管该子网的dhcp功能。由于这个两个`namespace`都是针对特定的子网创建的，因而在现有的`OpenStack`系统中，它们常常是成对出现的。

## 网络节点：外部网桥（br-ex）的网络
当数据从router中路由出来后，就会通过L、K传送到`br-ex`这个虚拟网桥上，而`br-ex`实际上是混杂模式加载在物理网卡上，实时接收着网络上的数据包。至此，我们的计算节点上的VM就可以与外部的网络进行自由的通信了。当然，前提是我们要给这个VM已经分配了`float-ip`。

关于OpenStack Neutron的分析大致描述这些吧，后续的话会详细的写一些文章来详细介绍整个的详细流程和相关的实现方式。


[1]:http://openstack.redhat.com/Networking_in_too_much_detail
[2]:http://www.ibm.com/developerworks/cn/linux/l-virtual-networking/
[3]:http://vtun.sourceforge.net/tun/faq.html
[4]:http://docs.openstack.org/network-admin/admin/content/under_the_hood_openvswitch.html