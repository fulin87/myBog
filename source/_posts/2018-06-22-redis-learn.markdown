---
layout: post
title:  "《redis设计与实现》读书笔记"
date:	2018-06-22 13:07:11 +0800
categories: 所思所想
---

> 这里记录一下阅读《redis 设计与实现》一书的读书笔记。

* 本书的官方网站，看了一眼，内容很丰富啊 **[本书官网](http://redisbook.com)**

## redis的数据结构的实现机制

### 简单动态字符串

redis的字符串对象的数据结构：
	
	struct sdshdr{
		int len; 	//记录buf数组中已经使用的字节的数量,等于字符串的长度
		int free; 	//记录buf数组中未使用的字节的数量
		char buf[]; //字节数组，用于保存字符串
	}

redis中字符串的优点：

* 获取字符串长度的复杂度是O(1)
* API是安全的不会造成缓冲区溢出
* 修改字符串长度N次最多需要执行N次内存重分配
* 可以保存文本或者二进制数据
* 可以使用一部分C字符串的函数

### redis中的链表

* 链表提供了高效的节点重排能力和顺序性的节点访问方式还提供了高效的增删节点能力
* redis中的列表，发布与订阅，慢查询，监视器都用到了链表
* redis本身还是要链表来保存多个客户端的连接状态
* redis还使用链表还构建客户端输出缓冲区

redis中链表的数据结构

redis中使用了两种hash算法，分别是**djb** 和 **murmurhash**

	typedef struct listNode{
		//前置节点
		struct listNode * prev;
		//后置节点
		struct listNode * next;
		//节点的值
		void * value;
	}listNode;

redis中list的数据结构

	typedef struct list{
		//表头节点
		listNode * head;
		//表尾节点
		listNode * tail;
		//链表所包含的节点数量
		unsigned long len;
		//节点值复制函数
		void *(*dup)(void *ptr);
		//节点值释放函数
		void (*free)(void *ptr);
		//节点值对比函数
		int (*match)(void *ptr,void *key);
	}list;

![](/content/image/redis_list1.PNG)

redis的链表实现是双端链表,而且是无环链表，redis链表可以保存各种不同类型的值

### 字典

哈希表的节点：

	typeof struct dictEntry{
		//键
		void *key;
		//值
		union{
			void *val;
			uint64_tu64;
			int64_ts64;
		} v;
		//指向下一个哈希表节点，形成链表
		struct dictEntry *next;
	} dictEntry;

* key属性保存着键值对中的键
* v属性保存着键值对中的值
* 值可以是一个指针，或者是一个uint64/int64的整数
* next属性是指向另一个哈希表节点的指针,依次来解决哈希冲突

redis字典的数据结构

	typedf struct dict {
		//类型特定函数
		dictType *type;
		//私有数据
		void *privdata;
		//哈希表
		dictht ht[2];
		//rehash索引，当rehash不在进行时，值为-1
		int rehashidx;
	}

* ht属性是一个包含两个项的数组，数组中的每一个项都是一个dictht哈希表
* 一般情况下，字典只使用ht[0]哈希表，ht[1]哈希表只会对ht[0]哈希表进行rehash时使用。
* rehashidx记录了rehash目前的进度,如果没有在进行rehash则其值为-1
* redis使用MurmurHash2算法来进行键的哈希运算

![](/content/image/redis_dict1.PNG)

* redis的rehash使用的是渐进式rehash
* 每次对字典进行curd操作时，程序除了执行指定的操作之外，还会顺带执行一部分rehash操作
* 最终在某个时间点上ht[0]的键值对会被rehash到ht[1]
* 渐进式rehash的好处在于它采取分而治之的方式，将rehash的操作分摊到对字典的每次操作上
* 渐进式rehash避免了集中式rehash而带来的庞大计算量

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


## redis集群

### redis的主从复制

![](/content/image/redis_sync.PNG)

	slaveof 192.168.182.128 6379   	//可以实现当前机器成128：6379这台机器的从服务器
	slaveof no one					//取消当前机器的主从同步关系

**注意**

	主服务器需要配置 bind:0.0.0.0 才能监听所有的ip过来的请求

redis的复制功能包括同步（sync)和命令传播(command propagate)两个操作

	同步：是将从服务器的状态更新成主服务器的状态
	命令传播：可以理解成是增量同步

同步的过程：
	
	1，从服务器向主服务器发送sync命令
	2，主服务器收到sync命令执行BGSAVE命令，生成RDB文件，并使用一个缓冲区记录从现在开始执行的所有写命令
	3，主服务器生成RDB文件完毕会向从服务器发送这个文件，从服务器载入这个文件，将自己的数据更新成和主服务器完全一致
	4，主服务器将缓冲区中的所有写命令发送给从服务器，从服务器执行这些命令

命令传播过程：
	
	主服务器需要将自己的写命令发送给从服务器，由从服务器执行

**注意**

	SYNC是一个非常耗费资源的操作
	1，主服务器执行BGSAVE命令会耗费大量的CPU，内存和IO资源
	2，主服务器发送RDB文件耗费大量的带宽和流量
	3，从服务器载入RDB文件会阻塞所有的名请求

**Redis2.8 以后版本的复制功能**

	1，使用PSYNC代替SYNC
	2，PSYNC具有完整同步和部分同步的功能
	3，完整同步是在初次同步的时候，和SYNC没有什么区别
	4，部分同步用于在从服务器断线后的同步

部分同步的实现依赖三个重要的概念

	1，复制偏移量，可以理解为是命令传播的字节数据量的总大小
	2，复制积压缓冲区，主服务器维护的一个固定长度的FIFO队列
	3，服务器运行ID

为了实现从服务器在断线之后和主服务器的数据一致性，需要进行同步，到底是全量同步还是部分同步，Redis主服务器依靠的是*复制积压缓冲区*来实现的

	1，复制积压缓冲区是一个固定大小的先进先出的队列，默认大小是1M
	2，主服务器进行命令传播时会将命令发送给从服务器，同时会将命令写入复制积压缓冲区
	3，复制积压缓冲区不仅会保存命令还会记录每个字节的复制偏移量
	4，当从服务器重新连上主服务器时，通过PSYNC命令会把自己的复制偏移量发送给主服务器
	5，主服务器会根据从服务器的复制偏移量来决定使用哪一种同步操作
	6，如果偏移量之后的数据还在复制积压缓冲区中则只需部分同步，否则执行全量同步

**注意**

复制积压缓冲区的大小的设计原则：

	复制积压缓冲区的大小  =  2 * second * write_size_per_second
	
	second : 从服务器断线后重新连接主服务器所需的平均时间，单位秒
	write_size_per_second : 主服务器平均每秒产生的写命令数据量（单位是M)

## redis的高可用方案

	由一个或多个sentinel实例组成的Sentinel系统可以监控任意多个主服务器和从服务器，并在