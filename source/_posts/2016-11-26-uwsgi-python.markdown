---
layout: post
title:  "nginx+uwsgi+python+mysql+flask 实战过程"
date:	2016-11-26 13:07:11 +0800
categories: python
---


> 使用python进行web开发，环境的搭建并不简单，这里面出现了很多问题。记录一下自己搭建环境的过程，顺便记录一下自己对于用python进行web开发的原理的一点粗浅的理解。

### nginx,uwsgi,python工作的原理

 如下图所示：

 ![](/image/python8.PNG) 

 以上是python进行web开发的基本工作原理，当然了，也可以不用使用uwsgi,但是使用uwsgi才是主流，非主流的这里就不记录了。

 * 用户发起http请求，请求到达nginx服务器。
 * nginx根据请求的url和自己的配置来决定怎么处理
 * 一般如果是静态请求，直接索引文件系统返回静态资源
 * 如果是动态请求，则交给后面的uwsgi来处理
 * uwsgi进程接到请求后，启动python解释器来执行python程序。
 * 执行解释返回结果

 以上是大体的过程，用词可能不够准确。这个过程中有一个核心概念：`uwsgi`，如果要把这个说清楚，会牵扯出一些列的知识：CGI,FAST-CGI,WSGI。这个之前有记录。

### 参考文档
	
[uwsgi官方文档英文](http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html)

[uwsgi官方文档中文](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/WSGIquickstart.html)



### 原生环境搭建过程

 *首先不使用任何python框架搭建一个原生的python web环境*

 我这里有一台虚拟机:

	192.168.116.131

 我将nginx和uwsgi都安装在131上，这里nginx我之前已经安装过了。

 * 安装uwsgi，安装到`/application/uwsgi-2.0.14`这个目录

		wget https://projects.unbit.it/downloads/uwsgi-latest.tar.gz
		tar zxvf uwsgi-latest.tar.gz -C /application/uwsgi-2.0.14
		cd /application/uwsgi-2.0.14
		make
 
 * 将uwsgi添加进环境变量搜索路径中

		cp /application/uwsgi-2.0.14/uwsgi /usr/local/sbin/

 * 建立一个软链接,这是一个好习惯

		ln -s /application/uwsgi-2.0.14 /application/uwsgi

 * 编写python脚本，这里假设mysql已经部署好了

		cd /home/fulin/temp/
		vi app.py
		
		#!/usr/bin/env python
		# -*- coding: utf-8 -*-
		
		"""application.py"""
		
		import MySQLdb
		def application(environ, start_response):
		"""Simplest possible application object"""
		start_response('200 OK', [('Content-Type','text/html')])
		conn = MySQLdb.connect(host = '10.2xx.xx.62',\
		                       port = 3306,\
		                       user = 'xx',\
		                       passwd = 'xxx',\
		                       db = 'test')
		cur = conn.cursor()
		
		aa =cur.execute("select * from vc_admin limit 10")
		x = []
		info = cur.fetchmany(aa)
		for i in info:
		    x.append(i)
		cur.close()
		conn.commit()
		conn.close()
		return '.'.join([str(i) for i in x])

 * 启动uwsgi

		uwsgi --socket 127.0.0.1:3031 --wsgi-file /home/fulin/temp/app.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

 * 配置nginx，因为这里仅仅是学习，所以不必用配置太复杂，先从最简单的开始

		location / {
		   include uwsgi_params;
		   uwsgi_pass 192.168.116.132:3031;
		}

 * 启动nginx

		cd /application/nginx/sbin
		./nginx

* 测试

		浏览器输入  http://192.168.116.131
		响应出admin表中的数据，表示OK

### uwsgi和flask进行集成

 `这里web服务器和应用服务器选择使用socket通信,不再使用低效的http`

 * 启用`ini`配置文件,文件为`/home/fulin/temp/web/config.ini`
	
		[uwsgi]
		socket = /tmp/my.sock     	#和web服务器通信的socket文件
		chmod-socket = 666		  	#更改socket文件的权限为666
		daemonize = /home/fulin/temp/flask2.log  #将uwsgi以守护进程的方式启动，同时制定日志文件
		master = true				#是否为主进程
		pidfile = /home/fulin/temp/master.pid   #创建主进程的pid文件
		wsgi-file = app.py	  #web项目的入口
		callable = app			#入口对象
		processes =1    		#进程数
		threads = 1				#线程数

 ***注意这里的ini配置文件出现是有背景的，之前我们启动uwsgi是使用命令行的方式：uwsgi --socket 127.0.0.1:3031 --wsgi-file /home/fulin/temp/app.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191，这种方式非常的不人性，如果能把命令行中的参数写入配置文件就好了。uwsgi是支持的。最标准的方法是使用ini格式的配置文件,我们的配置文件名为config.ini***

 ***这里要特别提醒一下,config.ini文件的开头的`[uwsgi]`这几个字符是不能少的，我就是因为没有这几个字符，启动的时候出现了一个很诡异的异常而浪费了很长时间***

 * 准备flask项目文件`/home/fulin/temp/web/app.py`

		from flask import Flask
	
		app = Flask(__name__)
		
		@app.route('/')
		def index():
		    return '<h1>hello world</h1>'
		
		@app.route('/<name>')
		def user(name):
		    return '<h1>hello, %s</h1>' % name
		
		#if __name__ == __name__:
		#    app.run(debug=True)

 ***这里要注意：`__name__==__name__`那两句代码是被注释掉的。因为uwsgi启动的时候会运行app.py，这段代码不注释掉，会启动python自己的 wsgi 服务器，从而产生冲突。也只有和uwsgi配合使用的时候才需要注释，开发的时候，还需要用这个功能才测试。***

 * uwsgi的启动

		uwsgi --ini /home/fulin/temp/web/config.ini
		输出：
		[uWSGI] getting INI configuration from /home/fulin/temp/web/config.ini
 
 * nginx的配置

		location / {
	       include uwsgi_params;
	       uwsgi_pass unix:/tmp/my.sock;
	    }

 ***这里要注意`uwsgi_pass unix:/tmp/my.sock;`,/tmp/my.sock这个文件，nginx进程需要有读写和执行权限才行，这个套接字文件是nginx和uwsgi进行通信的纽带***

 * 启动nginx

		cd /application/nginx/sbin
		nginx

 * 测试

		
		浏览器输入  http://192.168.116.131
		响应出hello world，表示OK

> 这两个环境的搭建，虽然逻辑比较简单，但是还是花了我两天的时候，我自己反思了一下，为什么会这么久，主要是有两方面的原因：一方面是之前确实没有接触过`uwsgi`,所以需要摸索一下，另一方面出现问题之后，没有认真看官方文档，很多细节，官方文档中都有详细的描述。如果认真阅读会事半功倍的。
 
  