---
layout: post
title:  "Elasticsearch实践总结"
date:	2019-03-20 13:07:11 +0800
categories: 存储
---

> 我最早听说 **ElasticSearch** 是在2014年初，记得当时还是在华为做外包，在开会的时候，有人提出有一个数据查询的需求做不到，原因是数据库的单表数据量达到了2千万，儿查询又需要多表关联。这是，架构师提出使用ElasticSearch来做。

### 索引

存储数据到ES的行为就叫 **索引**