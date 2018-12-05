---
layout: post
title:  "搭建自己的代理服务器"
date:	2018-08-16 13:07:11 +0800
categories: life
---

> 为了使用科学的上网，我自己搭建了一个Shadowsocks代理服务。这里记录一下自己搭建的过程

### 首选需要有一台位于国外的主机

 这里我选择的是国外的虚拟机,使用的是[云服务器厂商](https://my.vultr.com)的云服务,之所以选择这个是因为它支持支付宝支付。其他的需要使用信用卡的厂商，我总感觉不安全

### 安装 Shadowsocks 服务
		
		$ yum update    #这个命令的作用是更新一个yum源
		$ git clone https://github.com/flyzy2005/ss-fly

### 启动

		$ ss-fly/ss-fly.sh -i 123456 1024  #这里的123456是密码,1024是代理端口

### 配置

		默认的配置文件位于 /etc/shadowsock.json

### 启动与停止

		停止ss服务：ssserver -c /etc/shadowsocks.json -d stop
		启动ss服务：ssserver -c /etc/shadowsocks.json -d start
		重启ss服务：ssserver -c /etc/shadowsocks.json -d restart

> 使用了一段时间之后，发现vpn的速度不理想，看视频非常慢，于是找了各种办法进行加速，最终发现最简单快捷的加速方案还是bbr加速，以下是bbr加速的相关知识

### 原理

TCP BBR是google推出的TCP拥塞控制算法，目的是在有一定丢包率的网络链路上尽量跑满带宽，适合高延迟，高带宽的网络链接。

### 安装

	yum update  #更新yum源

	cat /etc/redhat-release   #查看系统版本

	#安装elrepo并升级内核
	rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
	yum --enablerepo=elrepo-kernel install kernel-ml -y

	#设置并重启系统
	grub2-set-default 0
	reboot

	#重启完成后检查内核是否已经是4.14版本
	uname -r

	#开启bbr:
	vim /etc/sysctl.conf    # 在文件末尾添加如下内容
		net.core.default_qdisc = fq
		net.ipv4.tcp_congestion_control = bbr

	#加载系统参数，输出了我们添加的那两行配置代表正常
	sysctl -p
		v6.conf.eth0.accept_ra = 2
		net.corenet.ipv6.conf.all.accept_ra = 2
		net.ip.default_qdisc = fq
		net.ipv4.tcp_congestion_control = bbr

	#确定bbr已经成功开启
	sysctl net.ipv4.tcp_available_congestion_control
		net.ipv4.tcp_available_congestion_control = bbr cubic reno

	lsmod | grep bbr
		tcp_bbr                20480  2

	

现在好了，可以看youtube的1080p的视频了，爽！！！

	
