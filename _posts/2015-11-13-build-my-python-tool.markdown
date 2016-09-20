---
layout: post
title:  "打造自己的python开发环境"
categories: fulin
---

* Table of Contents
{:toc}


## python的包管理工具pip
	1,访问：[get-pip.py](https://bootstrap.pypa.io/get-pip.py) 访问需要的python文件。
	2,将源代码copy到一个文本文件，将文本文件重命名的get-pip.py。
	3,通过命令行进入上述文本文件的所在位置，运行一下命令： 
    `python get-pip.py`
    

## pip工具的使用

	pip list              //查看已经安装的python包
    pip show package      //查看packageName的详细情况
	pip install package   //安装包package
	pip uninstall package //卸载包package
   

## python的命令行开发工具ipython

{% highlight shell %}
 python ipython  //安装ipython
{% endhighlight %}



***很多时候，配置一个开发环境不容易，这是一个学习的过程，也是一个过坑的过程，把这些过程记录下来，战胜自己的遗忘***