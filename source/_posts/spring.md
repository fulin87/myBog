---
layout: post
title:  "spring核心"
date:   2020-01-31 20:07:11 +0800
categories: java
---

> spring现在已经成为了java开发的标配，相当于事实上的"标准"。这里总结一下自己对spring的认识

###  spring最成功的地方在于抽象

spring将复杂重复的工作封装在框架内部，堆外暴露出了简介抽象的接口

* Resource ，资源
* BeanDefnitonReader , bean定义读取器
* BeanFactory，bean工厂



###  spring中的设计模式使用的出神入化

IOC容器本身其实就是一个工厂