---
layout: post
title:  "java中异常和JVM关系"
date:	2014-09-10 13:07:11 +0800
categories: 编程语言
---

> 经常听到有人说：系统宕机了，因为OOM导致的，那么OOM真的会直接导致JVM进程退出吗？

### JVM进程退出和异常的关系

- JVM进程退出会发生在以下场景

  - JVM进程中没有任何非守护进程在运行会导致JVM进程退出
  - 在某一个java线程中显示调用了System.exit()返回，会导致JVM进程终止
  - 在某一个java线程中显示调用了Runtime.getRuntime().exit()，会导致JVM进程终止
  - 在某一个java线程中显示调用了Runtime.getRuntime().halt()，会导致JVM进程终止
  - 通过操作系统发生关闭信号，比如kill命令，会导致JVM进程终止

- JVM进程终止和异常没有直接关系

  >  为什么平时生产故障经常会有这样的情况：JVM进程终止了，然后排查的过程中会发现有OOM异常，这给人一种错觉：*OOM会导致 JVM 进程终止*，其实这完全是错误，根本不是这么回事

  java中Exception和Error都是Throwable的子类，都是可以被try catch语句进行处理的。Throwable的来源有两种：*JVM抛出*  和 *程序中的throw语句抛出*

  - java中的异常都是线程自己处理的，不存在JVM来处理异常这种说法，不知道从哪里流传出来的说异常层层抛出最后是JVM来处理，这完全是错误的。
  - 线程处理异常的方式就是要么自己定义异常处理逻辑，要么终止线程。
  - 为什么OOM经常伴随这JVM的终止了？因为OOM发生后很大的概率导致线程纷纷死亡，最终JVM进程中没有非守护进程，从而导致JVM退出。

- 