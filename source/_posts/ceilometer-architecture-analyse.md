title: Ceilometer架构简要分析
date: 2014-08-12 20:13:46
categories: [OpenStack]
tags: [Ceilometer,OpenStack]
---
最近因为工作的需要以及论文的方向，需要了解OpenStack监控方面的知识。所以深入看了一下OpenStack的Ceilometer，大致分析了一下Ceilometer的实现机制和工作流程，因此也就形成了本文的对Ceilometer的一个大致框架介绍。

![](/img/2014/08/12/ceilometer-all-architecture.png)

<!--more-->

## 数据采集
Ceilometer的数据采集方式主要分为Poll和Push方式两种。

其中Push方式主要采集为OpenStack中各个组件模块中无法定时主动获取的事件消息，例如：虚拟机的创建，镜像的上传等等。该种方式的消息的采集依赖各个组件在事件发生时，依赖Ceilometer提供的消息机制将事件消息上报至消息队列当中。然后由Ceilometer-Collector中的notification-agent收集消息队列中的事件消息，然后交由指定的Pipeline将消息转换为指定的采样数据（Samples），转换之后的采样数据会被重新发送至消息队列当中，然后由Collector收集处理并存入数据库当中（MongoDB）。主要架构如下图：

![](/img/2014/08/12/push.jpg)

Poll方式主要采集OpenStack中的各个组件的统计数据和计算节点中的实时数据（该数据也是可以被随时统计获取的）。
在Controller节点上，Poll方式主要是启动相应的轮询进程（Pollsters)，依靠轮询进程定期调用组件模块的APIs获取各个组件的数据信息。然后将数据交由Pipeline进行处理，最后由Collector处理存储。此过程与上述Push方式一致。

![](/img/2014/08/12/central-poll.jpg)

在Compute节点上，Poll方式也是启动相应的轮询进程（Pollsters)，依靠轮询进程定期查询相应的信息，只是在数据采集方式上，采用虚拟机的相关驱动获取虚拟机的信息，目前主要的部署方案都是采用KVM-QEMU实现虚拟化，因此，底层信息获取上，采用的为LibVirt操纵虚拟机，同时也是通过LibVirt获取虚拟机的相关信息。当数据被采集之后，其之后的处理流程与上述两种方式都是一致的。

![](/img/2014/08/12/compute-poll.jpg)

## 数据处理
前面的数据采集工作完成之后，采集来的数据会交由Pipeline进行数据处理，Pipeline主要实现的是一个数据处理链的功能。Pipeline会根据不同的配置将0个或一个或多个Transformers组装成为一条数据处理链，在这条数据处理链的末端，会被装配一个Publisher。当数据进入这条数据处理链后，会被Transformers加工处理，然后由Publisher发送至消息队列当中，由Collector收集。

![](/img/2014/08/12/data-process.jpg)

Collector会时刻监听着消息队列，从消息队列中获取监控数据，然后将数据存入MongoDB中进行持久化。

![](/img/2014/08/12/data-save.jpg)

## 参考文档
1.[OpenStack System Architecture](http://docs.openstack.org/developer/ceilometer/architecture.html#high-level-description)
2.[ceilometer的数据采集机制](http://niusmallnan.github.io/_build/html/_templates/openstack/ceilometer_collect.html)
3.[OpenStack Ceilometer简介](http://www.cnblogs.com/alexyang8/archive/2013/02/18/2915981.html)
