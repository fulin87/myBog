---
layout: post
title:	"Linux中的文件大小和磁盘占有空间的关系"
date:	2016-10-27 13:07:11 +0800
categories:	linux
---

* Table of Contents
{:toc}

> 文件的真实大小和文件占有的磁盘空间不是一回事，根本的原因是由于系统和磁盘的存储机制决定的。最具有代表性的是crontab定时任务执行之后由sendmail产生的文件。下面详细分析一下整个过程。

## crontab 定时任务产生的问题

 现在有一台服务器A
	
	# cat /etc/redhat-release
	Red Hat Enterprise Linux Server release 5.4 (Tikanga)

	# rpm -qa|grep sendmail
	sendmail-8.13.8-2.el5

 是centos 5.4这个版本，而且已经安装了sendmail软件。

 有几个crontab定时任务
	# crontab -l

	0 * * * * /data/jobscripts/run_status60.sh
	0 * * * * /data/jobscripts/run_statusin.sh
	*/5 7-23 * * * /data/jobscripts/run_sendsms.sh
	*/5 * * * * /data/jobscripts/adjustment_product_store.sh
	0 3 * * * /data/jobscripts/collate_product_store.sh

 > **注意这里这里的定时任务都没有对标准输出和标准错误输出进行重定向到/dev/null文件，那么这样做有什么隐患呢？**

 定时任务启动之后，发现 `/var/spool/clientmqueue/` 目录下的文件开始不断增加，这是一年之后我无意中发现的情况：

	# ls -lihd /var/spool/clientmqueue/
	2080777 drwxrwx--- 2 smmsp smmsp 12M Oct 28 11:36 /var/spool/clientmqueue/

	# du -h /var/spool/clientmqueue/
	1.4G    /var/spool/clientmqueue/

 发现没有，目录 `/var/spool/clientmqueue/` 实际的大小只有12M，但是占有的磁盘空间确达到了1.4G，是不是非常奇怪？继续排查。

	# ls -lih /var/spool/clientmqueue/ | wc -l
	345047

	# ls -lih /var/spool/clientmqueue/ | head -5
	total 1.4G
	2080837 -rw-rw---- 1 smmsp smmsp  880 Jan 23  2014 dfs0NEf1nJ007902
	2080823 -rw-rw---- 1 smmsp smmsp  880 Jan 23  2014 dfs0NEK15v025165
	2080839 -rw-rw---- 1 smmsp smmsp  880 Jan 23  2014 dfs0NEp1UL009526
	2080836 -rw-rw---- 1 smmsp smmsp  880 Jan 23  2014 dfs0NEV2M6006278

 看到这里，才恍然大悟，原来 `/var/spool/clientmqueue/` 这个目录中的文件有345047个之多，但是每个文件却只有880 byte 大。这就造成了磁盘Block的大量浪费，所以才会有目录磁盘占用量远大于目录本身大小的情况。

 那么这些小文件是怎么来的呢？其实是crontab定时任务在执行之后，就会尝试通过sendmail给任务所有者发送一个邮件，如果发现sendmail服务没有启动，就产生了一个880 byte的小文件，临时放在`/var/spool/clientmqueue/`目录下，知道sendmail服务启动后才会清除。每次执行都产生一个，日积月累，数量非常庞大，给系统的安全还造成了一个隐患。

## 解决办法

 其实解决办法非常简单，将定时任务的标准输出和标准错误输出重定向到 `/dev/null` 即可，将定时任务改造如下：

	0 * * * * /data/jobscripts/run_status60.sh >/dev/null 2>&1
	0 * * * * /data/jobscripts/run_statusin.sh >/dev/null 2>&1
	*/5 7-23 * * * /data/jobscripts/run_sendsms.sh >/dev/null 2>&1
	*/5 * * * * /data/jobscripts/adjustment_product_store.sh >/dev/null 2>&1
	0 3 * * * /data/jobscripts/collate_product_store.sh >/dev/null 2>&1

 这里还有一个需要注意的地方，`/var/spool/clientmqueue/` 这个目录下面有太多的文件，如果需要删除,使用 `rm -rf /var/spool/clientmqueue/*`会发现删不了，因为文件太多，怎么办了，是有方法的。

 * 方法一

		ls /var/spool/clientmqueue/|xargs rm -f

 * 方法二

		find /var/spool/clientmqueue/ -type f|xargs rm -f

 * 方法三

		直接将目录   `/var/spool/clientmqueue/`	删除
		然后重建目录及给目录赋权限和属主和属组

	
## 启示

 通过这个案例，知道了crontab定时任务有一个小小的风险，但是可以通过重定向到 `/dev/null` 来解决。

 同时，对这个问题的深入剖析，可以帮助我们更加深刻的理解 Inode;Block 和文件存储的机制。