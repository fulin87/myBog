---
layout: post
title:  "《redis设计与实现》读书笔记"
date:	2018-06-22 13:07:11 +0800
categories: redis
---

> 这里记录一下阅读《redis 设计与实现》一书的读书笔记。

* 本书的官方网站，看了一眼，内容很丰富啊 **[本书官网](http://redisbook.com)**

## redis的数据结构的实现机制

### 简单动态字符串

	redis的字符串对象的属性有：
	
	int len; 	//记录buf数组中已经使用的字节的数量,等于字符串的长度
	int free; 	//记录buf数组中未使用的字节的数量
	char buf[]; //字节数组，用于保存字符串

redis中字符串的优点：

* 获取字符串长度的复杂度是O(1)
* API是安全的不会造成缓冲区溢出
* 修改字符串长度N次最多需要执行N次内存重分配
* 可以保存文本或者二进制数据
* 可以使用一部分C字符串的函数



redis是单进程单线程的K/V数据库,其为什么会那么快，主要是因为：

* 完全基于内存
* 数据结构简单
* 采用多路I/O复用

redis的数据淘汰机制

* redis可以通过配置文件设置maxmemory，即是最大内存使用容量
* 当可使用内存不足的时候开始进行数据淘汰策略
* redis的数据淘汰策略有6种

redis和memcache的区别

* memecache把数据全部存在内存中，没有持久化机制,Redis支持数据持久化
* memecache只支持String结构，Redis有比较丰富的数据结构
* value的大小，redis可以达到1G，memcache只有1M


