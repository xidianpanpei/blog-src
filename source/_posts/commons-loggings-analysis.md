title: commons-logging 源码简析
date: 2013-09-26 09:21
categories: [学习]
tags: [开源,commons-logging]
---

## 日志系统简介
应用程序中使用日志功能能够方便的调试和跟踪应用程序任意时刻的行为和状态。在大规模的应用开发中尤其重要，毫不夸张的说，日志系统是不可或缺的重要组成部分。由于开源的广泛性，我们不需要再去重复造轮子，我们可以直接使用众多的开源日志系统来满足我们的开发需求。

目前使用最多的日志系统主要有slf4j、commons-logging这样的“门面”日志系统，也有log4j、logback这样的实际执行日志系统。slf4j、commons-logging这类系统它们本身并不是直接实现具体的日志打印逻辑，而只是作为一个代理系统，接收应用程序的日志打印请求，然后根据当前环境和配置，选取一个具体的日志实现系统，将真正的打印逻辑交给具体的日志实现系统，从而实现应用程序日志系统的“可插拔”，即可以通过配置或更换jar包来方便的更换底层日志实现系统，而不需要改变任何代码。

出于对日志系统的深入学习，我们这次对commons-logging进行源码分析。

## commons-logging源码分析
我们对下载得到的commons-logging源码进行展开，很容得到commons-logging代码的主要代码结构。

<!--more-->

![](/img/2013/09/26/commons-logging-source-archeture.png)

其中的，部分代码文件已经作废，至于为什么没有删除掉，这就不得而知了。其中的LogSource.java文件已经作废，而其功能职责则由LogFactory.java代替。LogFactory.java实际只是一个实现接口，具体的实现则是由继承LogFactory.java的LogFactoryImpl.java来完成了。目录中众多其他实现文件则是继承自Log.java接口，对其实现了不同的功能，完成底层的对不同的实际执行日志系统的适配。主要是配有：AvalonLogger、Jdk13LumberjackLogger、Jdk14Logger、Log4JLogger、LogKitLogger、SimpleLog。还有一个ServletContextCleaner.java则是为了WebApp相关项目而准备，用于释放有关的内存。

![](/img/2013/09/26/jcl_class_diagram.jpg)

在LogFactory中定义了一定的规则，从而根据当前的环境和配置取得特定的Log子类实例。

Commons Logging中默认实现的LogFactory（LogFactoryImpl类）查找具体Log实现类的逻辑如下：  
1.查找在commons-logging.properties文件中是否定存在以org.apache.commons.logging.Log或org.apache.commons.logging.log(旧版本，不建议使用)为key定义的Log实现类，如果是，则使用该类。  
2.否则，查找在系统属性中（-D方式启动参数）是否存在以org.apache.commons.logging.Log或org.apache.commons.logging.log(旧版本，不建议使用)为key定义的Log实现类，如果是，则使用该类。  
3.否则，如果在classpath中存在Log4J的jar包，则使用Log4JLogger类。  
4.否则，如果当前使用的JDK版本或等于1.4，则使用Jdk14Logger类。  
5.否则，如果存在Lumberjack版本的Logging系统，则使用Jdk13LumberjackLogger类。  
6.否则，如果可以正常初始化Commons Logging自身实现的SimpleLog实例，则使用该类。  
7.最后，以上步骤都失败，则抛出LogConfigurationException。  

我们在使用Commons-Logging的时候，一般都是通过LogFacotry获取Log实例的，然后调用Log接口中相应的方法，而这是一个典型的工厂设计模式。Commons-Logging的实现可以分成以下几个步骤：  
**1.LogFactory类初始化：**  
a.缓存加载LogFactory的ClassLoader（thisClassLoader字段），出于性能考虑。因为getClassLoader()方法以后会使用AccessController，因而缓存起来以提升性能。  
b.初始化诊断流。读取系统属性org.apache.commons.logging.diagnostics.dest，若该属性的值为STDOUT、STDERR、文件名。则初始化诊断流字段（diagnosticStream），并初始化诊断消息的前缀（diagnosticPrefix），其格式为：“[LogFactory from ]”, 该前缀用于处理在同一个应用程序中可能会有多个ClassLoader加载LogFactory实例的问题。  
c.如果配置了诊断流，则打印当前环境信息：java.ext.dir、java.class.path、ClassLoader以及ClassLoader层级关系信息。  
d.初始化factories实例（Hashtable），用于缓存LogFactory（context-classloader –-< LogFactory instance）。如果系统属性org.apache.commons.logging.LogFactory.HashtableImpl存在，则使用该属性定义的Class作为factories Hashtable的实现类，否则，使用Common Logging实现的WeakHashtable。若初始化没有成功，则使用Hashtable类本身。使用WeakHashtable是为了处理在webapp中，当webapp被卸载是引起的内存泄露问题，即当webapp被卸载时，其ClassLoader的引用还存在，该ClassLoader不会被回收而引起内存泄露。因而当不支持WeakHashtable时，需要卸载webapp时，调用LogFactory.relase()方法。  
e.最后，如果需要打印诊断信息，则打印“BOOTSTRAP COMPLETED”信息。  

**2.查找LogFactory类实现，并实例化：**  
当调用LogFactory.getLog()方法时，它首先会创建LogFactory实例(getFactory())，然后创建相应的Log实例。getFactory()方法不支持线程同步，因而多个线程可能会创建多个相同的LogFactory实例，由于创建多个LogFactory实例对系统并没有影响，因而可以不用实现同步机制。  
a.获取context-classloader实例。  
b.从factories Hashtable（缓存）中获取LogFactory实例。  
c.读取commons-logging.properties配置文件（如果存在的话，如果存在多个，则可以定义priority属性值，取所有commons-logging.properties文件中priority数值最大的文件），如果设置use_tccl属性为false，则在类的加载过程中使用初始化cache的thisClassLoader字段，而不用context ClassLoader。  
d.查找系统属性中是否存在org.apache.commons.logging.LogFactory值，若有，则使用该值作为LogFactory的实现类，并实例化该LogFactory实例。  
e.使用service provider方法查找LogFactory的实现类，并实例化。对应Service ID是：META-INF/services/org.apache.commons.logging.LogFactory  
f.查找commons-logging.properties文件中是否定义了LogFactory的实现类：org.apache.commons.logging.LogFactory，是则用该类实例化一个出LogFactory  
g.否则，使用默认的LogFactory实现：LogFactoryImpl类。  
h.缓存新创建的LogFactory实例，并将commons-logging.properties配置文件中所有的键值对加到LogFactory的属性集合中。  

**3.通过LogFactory实例查找Log实例（LogFactoryImpl实现）：**  
使用LogFactory实例调用getInstance()方法取得Log实例。  
a.如果缓存（instances字段，Hashtable）存在，则使用缓存中的值。  
b.查找用户自定义的Log实例，即从先从commons-logging.properties配置文件中配置的org.apache.commons.logging.Log（org.apache.commons.logging.log，旧版本）类，若不存在，查找系统属性中配置的org.apache.commons.logging.Log（org.apache.commons.logging.log，旧版本）类。如果找到，实例化Log实例  
c.遍历classesToDiscover数组，尝试创建该数组中定义的Log实例，并缓存Log类的Constructor实例，在下次创建Log实例是就不需要重新计算。在创建Log实例时，如果use_tccl属性设为false，则使用当前ClassLoader（加载当前LogFactory类的ClassLoader），否则尽量使用Context ClassLoader，一般来说Context ClassLoader和当前ClassLoader相同或者是当前ClassLoader的下层ClassLoader，然而在很多自定义ClassLoader系统中并没有设置正确的Context ClassLoader导致当前ClassLoader成了Context ClassLoader的下层，LogFactoryImpl默认处理这种情况，即使用当前ClassLoader。用户可以通过设置org.apache.commons.logging.Log.allowFlawedContext配置作为这个特性的开关。  
d.如果Log类定义setLogFactory()方法，则调用该方法，将当前LogFactory实例传入。  
e.将新创建的Log实例存入缓存中。  

**4.调用Log实例中相应的方法**  

通过对commons-logging的分析，可以发现其实现的思路相对来说还是比较简单的，源码的整体结构也非常清晰，代码的数量也不是很多，里面的注释也是十分的清晰明了。