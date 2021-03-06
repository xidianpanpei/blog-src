title: 基础算法之堆排序
date: 2013-08-15 18:46
categories: [算法]
tags: [算法,堆排序]
---

最近一直都是比较闲的状态中，毕竟还是在过暑假嘛。于是拿起那本自从买来就没看过的「算法导论」看看，以前在网上总是看见将这本书推荐给入门的新人，不得不说这真的有点坑人的。

今天才刚刚拿起来，直接忽略第一部分的所谓「基础知识」，其实是看到那么多非基础的知识一下在看不下去了。看了下「堆排序」的章节，顺便回忆了一下以前学习的「堆排序」的知识。

「堆」无外乎就是一个特殊的完全二叉树，其特殊性在于堆的每个子树的根节点的值为该子树中的最大值/最小值，对应的堆就定义为「大顶堆/小顶堆」。「堆排序」的时间复杂度为O(nlgn)，这在排序算法中还是一个不错的时间复杂度的。接下来就是用代码实现了一下「堆排序」，这是不得不说下「算法导论」中的伪码还是不错的，至少比严奶奶的那本「数据结构」要强多了，但是这样的伪码有点不利于代码的实现，估计是太过于简要了吧。

<!--more-->

我是用的Java实现，其实是找了网上的其他代码参照着书本的伪码改写的，其中还因为忘记了Java中没有引用这种东西的，导致调试代码调了好长时间。

```java
	package com.test;	

	/**
	 * @author crazylion
	 *
	 */
	public class SortTest
	{
		public void heapAdjust(int[] a, int i, int size)
		{
			int lchild = 2*i;
			int rchild = 2*i + 1;
			
			int max = i;
			
			if(i <= size/2)
			{
				if(lchild <= size && a[lchild] > a[max])
				{
					max = lchild;
				}
				
				if(rchild <= size && a[rchild] > a[max])
				{
					max = rchild;
				}
				
				if(max != i)
				{
					//因为Java没有引用，只能多写点了
					int tmp = a[i];
					a[i] = a[max];
					a[max] = tmp;
					heapAdjust(a, max, size);
				}
			}
		}
		
		public void bulidHeap(int[] a, int size)
		{
			for(int i = size/2; i >= 1; i--)
			{
				heapAdjust(a, i, size);
			}
		}
		
		public void heapSort(int[] a, int size)
		{
			bulidHeap(a, size);
			for(int i = size; i >= 1; i--)
			{
				//因为Java没有引用，只能多写点了
				int tmp = a[1];
				a[1] = a[i];
				a[i] = tmp;
				heapAdjust(a, 1, i - 1);
			}
		}
		
		public static void main(String[] args)
		{
			int[] heap = new int[11];
			for(int i = 1; i <= 10; i++)
			{
				heap[i] = (int) (Math.random()*1000);
				System.out.println("==>" + heap[i]);
			}
			
			SortTest sortTest = new SortTest();
			sortTest.heapSort(heap, 10);
					
			for(int i = 1; i <= 10; i++)
			{
				System.out.println("-->" + heap[i]);
			}
		}
	}
```