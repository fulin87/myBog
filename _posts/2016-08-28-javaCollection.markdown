---
layout: post
title:	"java集合备忘录"
date:	2016-08-24 13:07:11 +0800
categories:	java
---

* Table of Contents
{:toc}

> java集合的运用和认识的深入程度是一个java程序员水平的重要体现，但是要想把这一部分的内容理解透彻，深刻掌握是需要整理和总结的。以前总结过，但是没有形成文字，时间长了难免会忘记，现在记录在此。

###java集合的总体框架
![](http://i.imgur.com/j6gYadi.png)

java集合类框架有两个顶层接口,有多个子接口和抽象类，若干实现类。这些接口和类可以抽象的分成2类：一类是"序列",一类是"键值对"。其实就是下面这两类：

	java.util.Collection
	java.util.Map

这里我们不讨论抽象类，直接说实现类。    
Collection的具体实现分成两部分：List相关的和Set相关的。List和Set都是接口。两者最大的区别就是元素是否唯一。List是不唯一的，Set是唯一的。    
List的具体实现有下面几种:
	
	ArrayList
	LinkedList
	
* *ArrayList是一个数组队列，相当于动态数组。它是由数组实现的，随机访问效率高，随机插入，删除效率低*    
* *LinkedList是一个双向链表。它可以被当做堆栈，队列或者双向队列进行操作。随机访问效率低，但是插入、删除效率高*   
* *ArryList和LinkedList都不是线程安全的，想要线程安全，可以使用Collections.synchronizedList(List list)进行处理*
* *List当然还有其他的实现类，比如：Vector，Stack。这些因为比较*


	

