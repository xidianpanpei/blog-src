title: 百度系统工程师初面之经历
date: 2014-10-08 18:14:33
categories: [面试]
tags: [百度,面试]
---

国庆长假刚刚过去，早晨还是迷迷糊糊的，还没有从长假休息状态恢复过来。正当眯着眼犯困的时候，手机很不应景的震动起来。结果是一个陌生的号码，接起来后才知道对方是百度的面试官，说是要我安排时间面试一下。考虑上午已经前往实习公司了，所以和对方约定了下午2:00面试。

<!--more-->

中午和老大请了个假，让后匆匆赶回学校。因为面试官下榻的宾馆离学校比较近，所以中午在宿舍磨蹭了一会儿，到下午1:20才出门。步行了2公里左右就到了面试地点（真的好近啊！！！）。由于提前到了，于是给面试官发了个短信，很快面试官就出来把我引入面试房间，说是面试房间，其实就是面试官的宾馆房间。面试官年龄看上去并不是很大，人很nice。等到都坐定了之后，就开始准备面试了。

其实，这次根本没有想到还会有面试的机会，当时相当于霸笔了，而且当时觉得卷子答的很是糟糕，就认定了是无望了。面试官开始翻阅之前的考卷，问我那个Linux下用宏计算结构体中变量偏移的题目现在会不会了，真心不知道那个宏究竟是什么，然后坦然告知还是不知道。于是，又接着问fopen()和open()的区别，这个考完后回来留心看了一下，然后解释了下主要是有cache缓存的区别就过去了。

以上算是开场白的话，后面面试官一边翻阅我的简历，一边让我自我介绍一下，这就算是正式开始了。首先是问了一下项目经历，问的并不是很细，只是粗略的了解了一下。由于两个在实验室的项目因为各种原因都是不了了之结束的，所以还特意的解释了一下原因。由于自己一直是在做云计算的东西，尤其是在学校这段时间，一直都是在搞OpenStack相关的东西。所以当我介绍之前搞得一个关于OpenStack
Neutron的项目的时候，面试官让详细介绍一下，似乎很是感兴趣（后面事实证明他在这方面应该也在做相应的研究和开发）。我向他介绍了一下Neutron的机制，整个网络架构和数据通信关系。然后还介绍一下Neutron现有的缺陷，说是存在单点故障的问题，而且网络节点承担整个集群的网络流量存在很大的问题。于是扯到社区上刚提出来的DVR的问题，并把DVR解释了一边。结果面试官问我DVR解决的是东西向流量问题还是南北向流量问题，想了一会儿觉得是外部流量问题，所以就回答了南北向，结果被告知是东西向的。回来仔细想想，其实这个问题并不是很好回答，因为DVR的确是解决了东西向流经网络节点的流量，但也是解决部分外部网络访问的流量问题，只不过像面试官所说的，解决方案上的问题还是比较多而已。

接下来，又问了一个系统设计的问题，针对neutron上的DHCP，面试官提问如果把DHCP服务集中到单一节点上服务，应该如何设计解决，想了一下后，就说要在每个计算节点上加入一个agent来监听每个ovs网桥，并在DHCP节点使用namespace隔离dhcp-server服务，并做好相应的持久化功能。对于这个答案，面试官也没有多说。然后又问了下OVS（OpenVSwitch）上的接口有多少中类型，只回答了两种。

算法题要求用python写一个字符串逆序的函数，不能用内置的reverse函数，然后就是一个二进制数的最大匹配，就是给定大于10w个(0, 2^32)的随机不重复的数，然后给定一个整数，求与其二进制匹配最长的数。先是给了一个求模递归的解法，效率较低，然后又改进使用异或方式给了个线性复杂度的解法。

在语言方面，由于本人的C功底不行，然后就只问了python中的类继承的问题，其实就是子类寻找父类的遍历方式，当时的答案被面试官说是错的，可是回来试试之后，觉得自己当时的回答应该是没错的，汗！！！

其中还穿插一些OpenStack的一些细节问题，一时也无法一一回想描述了。感觉这次面的一般般，不知道是不是要炮灰的节奏！
