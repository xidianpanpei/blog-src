title: 利用Python简单搜索人人网好友关系
date: 2013-09-08 17:23
categories: [Python]
tags: [爬虫,Python]
---

![](http://ww1.sinaimg.cn/large/7458d655gw1f8mfyjqy7xj20dw09uq4q.jpg)

最近乘着开学的时候还没有什么事情，于是乎，开始学起了Python。Python作为脚本语言，最近一段时间真的是越来越火了，所以也有必要认真的学习一下的。毕竟多学一门语言也不是坏事，而且看上去，Python的学习门槛不是太高，只是在一些库上的学习和应用值得花些时间好好钻研一下。

<!--more-->

不得不说Python由于去掉很多的数据类型，封装了很多的功能，加上那丰富的库，真的让开发变得很是容易，代码量更是大大的减少了。然而，并不是像很多其他人那样，总是推荐新手去学习Python，我个人认为如果只是玩玩编程而已，Python的确是个不错的选择，但是如果想要深入学习的话，还是拿C这类的语言来入门吧，虽然门槛要高一些，但是更能去理解程序的执行原理(当然，汇编之类的更是容易了解程序的执行原理，但大多数人还是愿意用稍微高级些的语言，毕竟我们不是机器人)，而且以后学习其他的语言也是更容易一些入门。

废话不多说了，这些也就是顺便一提。

爬取人人网上的好友信息也就是看到别人有弄过，觉得也挺好玩的。之前从来没有使用开放平台的`APIs`开发过的经验，于是也就是在人人上注册了开发者的账号，获取到`accesskey`，然后拿着那刚开始入门的Python开始胡乱的写了起来。我的目的也是很简单的，就是写个简单的爬虫，爬点人人网上的好友信息玩玩。想来想去，也就好友之间的关系图看起来挺有意思的，于是找到相应的接口，胡乱的玩起来，而主要就是根据一个好友的id，然后搜索到其所有的好友id，然后是好友的好友的id，如此重复下去。理论上来说，这样一直爬下去的话，应该是可以爬完整个人人网上的id(那些僵尸账号是爬不到的，孤立在整个网络之外的孤点估计都没有办法爬到的)，然后将好友写入文件或者数据库，有机会的话还打算用`d3.js`。刚开始是用好友关系列表中获取到相应的好友id，当时只注意到这个API。今天突然发现还有个直接获取好友ID列表的API，很当然就用了这个接口。

目前的这个简单的爬虫很是粗糙，虽然暂时能爬取好友列表，但是暂时还是无法解决关系网中存在的回环问题，其实这个问题只要进行id去重就好，可是利用内存表去重的话，怕是到后来随着id表的规模的扩大导致内存吃不消，同时性能更是糟糕。其实，可以数据库来去重的，可惜暂时不打算使用数据库。利用文件存储也是可以的，但是怕到时候频繁的I/O造成性能下降。总之，还是等到后面再想办法吧。还有个就是万恶的人人网的访问请求限制，今天刚跑到1w的关系量就被禁止了。

多说也是无益啊，还是先贴上粗糙的代码吧！

```python
#name:renren-friend.py
#--*utf-8*--

import Queue
import urllib2
import json
import time

def work(queue, datafile):
    url = "https://api.renren.com/v2/friend/list?"
    access_token = "access_token=put your access_token here"
    id = queue.get()
    userId = "userId=%d" %id
    #pageSize设置为7000是为了一次性获取所有好友ID，人人网的VIP好友上线是7000
    data = url + access_token + "&" + userId + "&pageSize=7000"

    response_text = urllib2.urlopen(data)
    #睡眠5秒，为了防止人人网的访问频繁的限制，事实证明还是太低了，估计得设置到2分钟
    time.sleep(5)
    s = response_text.read()
    response = json.loads(s)

    r = response['response']
    length = len(r)

    for i in range(length):
        queue.put(r[i])

        print "---> %d" %r[i]

        strword = "%d%s%d%s" %(id, " ", r[i], "\n")
        datafile.write(strword)

if __name__ == '__main__':
    fp = open("friends.txt", "w")
    queue = Queue.Queue()
    queue.put(100896521)

    while not queue.empty():
        work(queue, fp)

    fp.close()
    print "------end the search-------"
```

第一版的粗糙的代码暂时贴在这里吧，后面还是要花点时间继续改进，同时也要好好的学习Python的。
