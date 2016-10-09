title: Windows终端环境搭建
date: 2015-04-30 18:31
categories: [杂项]
tags: [Babun,终端,工作环境]
---

![](http://ww1.sinaimg.cn/large/7458d655gw1f8mezaqwutj21kw11z4qp.jpg)

最近已经入职了新的公司，不错的是公司给分配了配置还算不错的电脑。悲剧的是，操作系统是Windows7。之前使用Ubuntu的操作系统都已经快一年多了，都已经习惯了在Ubuntu下进行各个工作了，虽然在IM工具和Office工具上还是有些问题，但是影响还不至于很大。反倒是一下换回到了Windows环境下，反而有些不习惯了。

<!--more-->

因为之前一直在从事云计算相关的开发与研究，所以会经常登录各种服务器，Ubuntu下的Terminal使用起来真的是很爽很方便。然而在Windows下，那个DOS Terminal一下子让我感受到了MS深深地恶意。没办法，只能自力更生，自己想办法解决了。之前是知道Cygwin的，而且也知道Cygwin有着些不足，并不是特别想用。于是在网上继续搜索着，终于找到一个叫做[Babun](http://babun.github.io/) 的软件。这东西是基于Cygwin做了层封装，但是做的还是挺好的，关键是居然还有一个Package Manager，这个真的是很难得的，更难得的是居然还有不少的软件可以安装。

首先是从Babun的[官网](http://babun.github.io/)下载Babun的安装包，Babun 默认安装在 %USER_HOME%\.babun 目录，但是公司的电脑就C盘和D盘，而且C盘又小的可怜，没办法，只能将其安装到D盘了。解压Babun安装包，然后使用管理员权限打开cmd Dos界面，在安装包文件夹中执行以下命令即可：
```install.bat /t "D:\Babun"```

等到命令执行完毕，Babun也就顺利安装完成了，看上去是不是很简单的。但是刚刚安装完毕的Babun的界面还是只是经典Dos风格的，看上去还是很丑。然后没办法，我只能祭出之前在Ubuntu上修改Terminator的办法，使用Solarized Dark配色的配置方法加上dircolors将其修改的有点Linux终端的味道了。最后就是使用私家的[Vim配置](https://github.com/xidianpanpei/dot-vimrc) 调整好Vim。整个过程折腾下来还是有些麻烦的，但是总算能够在Windows上有一个可用的Terminal了。

最后祭上最终的Babun的截图，看上是不是蛮不错的！
![](http://ww2.sinaimg.cn/large/7458d655gw1f70ojtmx8lj21270kadlo.jpg)
