---
layout: post
title:	"windows软件图标异常的恢复方法"
date:	2016-10-29 13:07:11 +0800
categories:	杂项
---

> 作为一个有轻微强迫症的用户，遇到一点不协调的东西，都会有点不舒服，一周以前，我电脑中的outlook图标出现了异常，变成了很难看的样子，一直想把这个图标恢复，没有找到什么好的办法，咨询了很多人，也没有解决，这个事一直耿耿于怀，直到今天，才解决，这里记录一下解决办法......

## 异常之后的图标

 ![](/content/image/out_look_err.PNG)

## 解决办法

	1,切换至用户家目录中的本地数据目录，比如我的目录是：C：\User\870850\AppData\Local
	2,找到文件：iconcache.db
	3,将iconcache.db文件删除
	4,重启系统

## 原理

 `C：\User\870850\AppData\Local\iconcache.db` 文件是一个缓存的数据文件，存储着系统中应用程序的图标等信息。将其删除后，系统重启时在explorer.exe程序启动时，会重建这个文件，也就会解决图标异常的问题了。

## 解决之后

 ![](/content/image/out_look_right.PNG)

> 这下看着舒服多了，哈哈哈！！！！