---
layout: post
title:  "mybatis generator"
date:	2018-11-05 13:07:11 +0800
categories: java
---

> **mybatis generator** 可以大幅度的提升开发的效率，这里总结一下相关的知识点

###相关资源

**[generator官方文档](http://www.mybatis.org/generator/)**

**[generator源码](https://github.com/mybatis/generator)**

###generator到底是个什么东西？

*一句话，mybatis generator就是一个工具，可以实现从数据库表结构到java实体类和mapper映射文件之间的自动转换*
 

###generator的使用方式

* 通过ant使用
* 通过maven 插件的方式使用
* 以java编码的方式使用
* 以eclipse插件的方式使用

这里我们只用关注maven插件的使用方式

###generator的配置文件

generator配置文件的作用在官方文档中是这样描述的：

配置文件将告诉MBG

* 怎样连接数据库
* 创建哪些对象，怎样创建这些对象
* 哪些表将会用于对象的创建

