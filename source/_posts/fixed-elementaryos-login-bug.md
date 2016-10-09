title: Elementary OS循环登录问题
date: 2014-02-20 10:28:45
categories: [杂项]
tags: [ElementaryOS,登录]
---
![](/img/2014/02/20/elementaryos.png)

[Elementary OS](http://elementaryos.org/)自从出来之后，便声称是“最美的Linux”。我在很早之前看了相关介绍后，出于对于Elementary OS简洁精致的界面的喜爱，便很快安装了Luna版本的，这也是目前唯一的一个正式版本。

<!--more-->

由于一直把安装了Elementary OS的电脑当作服务器使用，便一直没有关机（实验室的电脑，因此才可以不关机啊）。等到寒假放假回家，实验室要求断电，这才把电脑关机。可是关机的总是在界面无法关机，当时也没有太注意，直接就给强行关机了。等到前两天回到实验室，再次开机时，才发现已经无法通过图形界面登录进去了，也就是tty7无法登录。通过Google的帮助，弄了小半天，搞定了界面登录问题。（其实主要是Elementary OS太过于小众，没有多少资料，不过由于其是基于Ubuntu的，可以使用一部分Ubuntu的资料）

### 问题描述
开机进入图形登录界面后，输入密码后系统开始确认，先是黑屏几秒，然后就直接跳回到登录界面，一直无法进入桌面。但是，使用`Ctrl`+`Alt`+`F1`，可以进入tty1，通过命令行登录进去。

### 解决办法
1.使用`Ctrl`+`Alt`+`F1`，使用命令行界面登录系统。  
2.执行命令`sudo su`，切换至`root`权限。  
3.执行命令`ls -lah`，可以看到：

    -rw-------  1 root root   53 Nov 29 10:19 .Xauthority

4.执行命令`chown username:username .Xauthority`，修改文件的用户和用户组，然后重启系统，就可以正常登录了。

### 参考资料
《Ubuntu gets stuck in a Login Loop》:[http://askubuntu.com/questions/223501/ubuntu-gets-stuck-in-a-login-loop](http://askubuntu.com/questions/223501/ubuntu-gets-stuck-in-a-login-loop)
