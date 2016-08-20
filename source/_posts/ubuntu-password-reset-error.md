title: Ubuntu密码重置无效错误
date: 2014-02-26 23:09:52
categories: [杂项]
tags: [Ubuntu,密码]
---
![](/img/2014/02/26/ubuntu.png)

## 问题描述
今天，实验室的一同学在登录自己的Ubuntu系统时，发现自己忘了登录密码。于是重启进入`恢复模式`中，想要重置用户密码，结果执行相关命令后，得到以下错误提示：

<!--more-->

    root@username-PC:~# passwd username
    Enter new UNIX password:
    Retype new UNIX password:
    passwd: Authentication token manipulation error
    passwd: password unchanged

我刚开始以为是同我上次的系统循环登录的原因是类似的，于是按照我的[教程](http://panpei.net.cn/2014/02/20/fixed-elementaryos-login-bug/)操作，修改了相关文件的权限，重新在`恢复模式`下重置密码，结果还是同样的错误。在伟大的Google的帮助下，找到了解决问题的办法，原因似乎是磁盘的根目录挂载出现了问题，执行以下命令：

    mount -rw -o remount /

然后在`恢复模式`中就可以重置密码了。

## 参考资料
《Authentication token manipulation error》:[http://askubuntu.com/questions/91188/authentication-token-manipulation-error](http://askubuntu.com/questions/91188/authentication-token-manipulation-error)