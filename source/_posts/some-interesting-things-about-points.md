title: 指针的一点理解
date: 2014-03-04 11:27:06
categories: [C/C++]
tags: [C语言,指针]
---
![](http://ww2.sinaimg.cn/large/7458d655gw1f8mfmzm62oj20m80m8t9q.jpg)

每天打开电脑的第一件事就是开QQ、开微博（躺枪的举个手），然后就是打开自己的InoReader（前几天还是Digg Reader，但是嫌弃界面太简洁和功能太简单了，就换了）。每天都能看到不同的有趣的文章其实也是中享受的。

<!--more-->

今天就在「伯乐在线」上看到一篇[文章](http://blog.jobbole.com/60647/)，其实算不上文章的，只是把StackOverFlow上面的一个问题给翻译了，但是内容本身还是值得探讨的。这个问题就是讨论指针的，没错，就是C/C++的指针的，估计很多人认为很简单了。但是说实话，我到现在还是不敢说自己已经完全了解指针的，当然也没有把指针玩的很溜的。

关于指针我记得有个经典的段子，形象的说明指针的优势和缺陷的，可惜已经记不太清楚了。个人认为，编程语言无非就是为我们提供一套操作内存与CPU的规则，而我们写出来的程序，就是我们利用这套规则编写的流程指导书而已。基于这种想法的话，我们会发现C语言提供的指针，真的是个非常美妙的工具。因为指针的存在，我们对于内存的操作，可以达到前所未有的灵活多变。当然，就像玩双节棍一样，在高手手中，是威力无穷的武器，但是到了低手手中，说不定就是自伤八百的凶器了。而低手就是要在这种近乎自残的锻炼方式中，才能慢慢成为高手。

记得当初学C语言的时候（相信大家都一样，大学学的第一门语言都是C语言），由于课时紧张，指针这个最重要的部分，直接就被授课老师给略掉了，所以后来一直对此耿耿于怀，因为自己走了好多弯路，才明白编程的意义所在。指针真的很重要，我认为只要我们真的理解指针的意义，对于编程来说，我们已经了解了一半了。然而，等我把指针弄懂了，其实指针本身也不难，但是却是难在如何使用上，对于指针的使用，其实就是我们对于内存的操作过程。当我们把指针调用来调用去的时候，很容易就把自己给绕晕了，最后连自己都不知道某个到底指到哪块内存了。所以，个人认为，使用指针最重要的原则就是，自己一定要有清晰的思路，能够明晰知道自己操作的指针的具体指向，否则就别用了吧。

说了这么多，很多其实都是废话了。其实，C中的指针对于我们来说，最终要的就是记住两个符号：\*和&，这两个符号会一路伴随着我们的指针使用历程的。\*表示取值，而&则表示取址，谨记它们的含义，那么我们的指针历程也就不会那么艰辛了。

关于单重指针，我们假设有以下的范例：

```
#include <stdio.h>

int main()
{
	int i = 5;
	int *p = &i;

	printf("i's address is: %lx\n", &i);
	printf("p's value is: %lx\n", p);

	printf("*p's value is: %d\n", *p);

	return 0;
}
```

我们简单编译执行后，就有了以下的结果：

```
i's address is: bfd42908  
p's value is: bfd42908	
*p's values is: 5
```

很明显啦，p就是指向i所在内存的一个指针，其实就是i的别名。回过头来我们在来看看\*，其实\*就是说明指针的符号，它表示p是一个指针变量，如果是两个\*\*则是表示指针的指针，然后我们可以依次类推一下了。

我们再来看看二重指针，也就是那篇文章中讨论的问题。

```
#include <stdio.h>

int main()
{
	int i = 5;
	int j = 6;

	int *ip1 = &i;
	int *ip2 = &j;

	printf("i's address is: %lx\n", &i);
	printf("j's address is: %lx\n", &j);

	printf("\n");

	printf("ip1 points to: %lx\n", ip1);
	printf("ip2 points to: %lx\n", ip2);

	printf("\n");

	printf("ip1's address is: %lx\n", &ip1);
	printf("ip2's address is: %lx\n", &ip2);

	printf("\n");

	int **ipp = &ip1;

	printf("ipp points to: %lx\n", ipp);
	printf("ipp's address is: %lx\n", &ipp);

	printf("---------------------\n");

	*ipp = ip2;

	printf("ip1 points to: %lx\n", ip1);
	printf("ip1's address is: %lx\n", &ip1);

	printf("\n");

	printf("ipp points to: %lx\n", ipp);
	printf("ipp's address is: %lx\n", &ipp);

	printf("\n");

	printf("*ipp's value is: %lx\n", *ipp);
	printf("**ipp's value is: %lx\n", **ipp);

	return 0;
}

```

这段代码执行完成后，我们会有以下的结果：

```
i's address is: 22ff1c  
j's address is: 22ff18	

ip1 points to: 22ff1c	
ip2 points to: 22ff18	

ip1's address is: 22ff14	
ip2's address is: 22ff10	

ipp points to: 22ff14	
ipp's address is: 22ff0c

*ipp's value is: 22ff1c  
**ipp's value is: 5	
++++++++++++++++++++++++++ 
ip1 points to: 22ff18  
ip1's address is: 22ff14  

ipp points to: 22ff14  
ipp's address is: 22ff0c  

*ipp's value is: 22ff18  
**ipp's value is: 6
```

从上面的结果我们可以看到，\*\*ipp是一个二重指针，它实际上是指向了一重指针\*ip1的内存地址。也就是说ipp实际上指向指针ip1的指针。

没改变之前的最后的两行可以看到，\*\*ipp的值为5，也就说，ipp最终指向的还是变量i，而\*ipp也是存着ip1的地址值得，也就说ipp还是指向ip1的，而\*ipp实际上指针ip1的别名罢了。

OK，接下来我们来改变一次：`*ipp = ip2;`，可以发现ip1指向了j，而且\*ipp和\*\*ipp也都随着ip1的变化而变化了，这也再次证明了我们关于\*ipp的说明：\*ipp是ip1的别名。下面我们可以看看更为直观的图示：

![](/img/2014/03/04/a.gif)

![](/img/2014/03/04/b.gif)

因此，我们可以知晓多重指针中的\*的作用，当某个指针\*\*ipp的\*被去除了一个再次操作（\*ipp），其实就是对\*\*ipp指向的那个内存中的数据进行操作，而\*ipp也其实就是这块内存的另一个别名罢了。
