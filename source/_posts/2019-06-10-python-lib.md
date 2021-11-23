---
layout: post
title:	"python2和python3以及python常用工具包的那些事"
date:	2019-06-06 13:07:11 +0800
categories:	编程语言
---

> python 可以大幅度的提升我们工作的效率，自己写的一些python小工具非常实用，但是经常面临一个问题，自己的电脑环境换了之后python的包就需要安装，这时候搞一个环境就比较麻烦，这里总结一下自己经常用的一些包的安装和使用方法

## python2和python3

曾经看过一个段子：“python2和python3的关系，就是java和javaScript的关系”，虽然有点夸张，但是也充分说明了python2和python3之间的巨大差异。

同时使用python2和python3，我推荐使用py的方式，因为这是python官方推荐的方式

* **py -2 test.py**  #使用python2运行test.py
* **py -3 test.py**  #使用python3运行test.py
* **#! python2**     #脚本文件的最开始加上这个，然后 **py test.py** 表示使用python2运行test.py
* **#! python3**     #脚本文件的最开始加上这个，然后 **py test.py** 表示使用python3运行test.py

* **py -2 -m pip install package**  #表示安装Python2的package

* **py -3 -m pip install package**  #表示安装python3的package

* **py -3 -m pip install --upgrade pip**  #升级python3的pip

  ****



### kafka-python

* python的kafka客户的，可以非常方便的操作kafka
* py -3 m pip install kafka-python









## 我的 requirements.txt

~~~
kafka-python==1.4.7
redis==3.3.11
requests==2.22.0
pycrypto==2.6.1
~~~

