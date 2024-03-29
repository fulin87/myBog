---
layout: post
title:  "代码重构的思考"
date:	2019-05-01 13:07:11 +0800
categories: 所思所想
---

> 总结一下有关代码重构的思考和实践经验



###  为什么大部分系统都会越来越乱

这里所说的系统不仅仅是指软件系统，可以是一个抽象的宏观的概念。比如硬件系统，社会组织。

整理干净的房间，过一段时间就会变的凌乱不堪

一辆崭新的汽车，使用几年之后会变的老化，部件损坏,落满灰尘

一个团队，随着人数的增长，管理起来会越来越困难

那么到底是为什么呢？



###  热力学第二定律

这是 *定律* 不是 *定理* ，这是我们宇宙的根本法则。我上高中的时候，学到这个定律的时候，老师只是轻描淡写的进行介绍和解释，现在回想起来真是觉得可惜。这个定律最开始是热力学中的概念，指热量不会自发的从低温物体转移至高温物体，除非有外界做功。后来人们在此基础上进行了一系列的推广，用一句话说 *热力学第二定律* 也是 **熵增定律** ，熵 是近代物理学中的概念，指一个系统的 **混乱程度** 我第一次知道还是在大一的工程化学棵上，

这个定律不是某个人发现的，而是很多人根据一些列的试验和现象总结出来的，这个定律的牛逼之处是：预言了我们这个宇宙的终结宿命：归于寂静，说白了整个宇宙最后的结局就是：毫无生气，一片死寂，熵达到最大。

明白这一点真是让人难过，不过不用担心，因为相对宇宙的无限，我们的生命实在是太短暂了.....



### 软件系统变的难以维护的根本原因

明白了上面的热力学第二定律就知道为什么软件系统会变得难以维护。如果要使软件系统井井有条，优雅健壮，就需要对系统做功。

这里的做功，可以是清晰易懂的文档，严格的代码规范，简单高效的设计。当然做功是需要持续进行的，一旦停止，归于混乱只是时间问题。



### 代码重构的目的是什么？

代码重构的目的就是输出 **简单，清晰，易读** 的代码

