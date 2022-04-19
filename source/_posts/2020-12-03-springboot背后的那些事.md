---
layout: post
title:  "springboot背后的那些事"
date:   2021-12-03 20:07:11 +0800
categories: java
---

>  springboot并没有大部分人想象的那么简单，它其实是把复杂度隐藏起来了,这里记录一下自己探索的springboot背后的那些细节

###  微服务到底是什么
微服务是一种风格（style),并不是一种标准,最早是 **[martinflow](https://martinfowler.com/)** 提出来的,martinflow的个人网站上有很多非常有价值的文1章，很值得一读，其提出的很多理念值得我们深入的思考。
![](/content/image/microservice.png)

* 微服务仅仅是一种软件架构风格
* 微服务并没有统一的标准,有优点,也有缺点
* 所谓的组件：就是一个软件单元，是独立的，可替换的，可升级的
  
关于微服务，马丁的这篇文章已经描述的非常详细了: **[microservice](https://martinfowler.com/articles/microservices.html)**

### 微服务和SOA的关系
SOA出现的比微服务要早很多，SOA也是一种架构风格，和微服务非常相似，微服务可以看做是SOA的一种演进结果。微服务更加注重敏捷，CI,CD,devOps

### springboot到底是怎样启动的
首先,springboot项目(spring-boot-maven-plugin插件)打包出来的jar包是怎样的一种结构呢？ 将打包出的jar包解压之后，是这样的

```
    BOOT-INFO
    META-INFO
    org 
```

### java应用的调试
JDWP是jvm的一个调试协议,是分析源码的有力武器,同时也可以用于开发环境bug的远程调试。

服务端启动
```
    java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5050 -jar springbootDemo.jar

    输出：
    Listening for transport dt_socket at address: 5050

    transport=dt_socket 表示服务启动的时候启动socket协议启动远程调试
    server=y 表示要监听着调试器对它的连接
    suspend=y 表示服务已启动就要等待调试器的调试
    address=5050 表示在5050端口等待调试器的连接
```

客户端启动
``` 
    java -anentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5050
```

### springboot启动的背后
springboot启动的时候会加载jar包中的META-INFO/spring.factories文件
根本就没有什么自动化，只不过都是springboot在背后做准备好了而已  

### spring源码背后的那些小细节
* spring内部的类名都非常长,其实这是非常值得借鉴的，因为spring用一个很长的类名，准确的刻画了类的含义,一个简写的类名可能只有开发者自己能看懂其含义