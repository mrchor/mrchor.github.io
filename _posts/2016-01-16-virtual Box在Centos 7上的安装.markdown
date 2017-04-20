---
layout:     post
title:      "virtual Box在Centos 7上的安装"
date:       2016-01-16 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Linux VirtualBox CentOS
---

> “这就是我，一个低调的作者。”



1、首先，我们需要在oracle官网下载virtual Box的centos7版本：

　　下载地址为：http://download.virtualbox.org/virtualbox/5.0.12/VirtualBox-5.0-5.0.12_104815_el7-1.x86_64.rpm

2、使用rpm安装virtualbox：

	rpm -ivh VirtualBox-5.0-5.0.12_104815_el7-1.x86_64.rpm
	
　出现错误，显示以下信息：
 
	 警告：VirtualBox-5.0-5.0.12_104815_el7-1.x86_64.rpm: 头V4 DSA/SHA1 Signature, 密钥 ID 98ab5139: NOKEY
	错误：依赖检测失败：
			libQtGui.so.4()(64bit) 被 VirtualBox-5.0-5.0.12_104815_el7-1.x86_64 需要
			libQtOpenGL.so.4()(64bit) 被 VirtualBox-5.0-5.0.12_104815_el7-1.x86_64 需要
			
这是因为有两个依赖包没有安装，我们需要安装以下，使用在线安装，执行两条命令：

	yum install qt
	yum install qt-x11
	
出现错误，显示以下信息：

	Recompiling VirtualBox kernel modules                      [失败]
	  (Look at /var/log/vbox-install.log to find out what went wrong)
	ln: 目标"setup" 不是目录
	
查看其中错误日志：

	cat  /var/log/vbox-install.log
	Makefile:185: *** Error: unable to find the sources of your current Linux kernel. Specify KERN_DIR=<directory> and run Make again。 停止。
	
从网上获知，是某些依赖软件没有更新到最新，需要更新：

	yum groupinstall "Development Tools"
	
再次执行安装：

	rpm -ivh VirtualBox-5.0-5.0.12_104815_el7-1.x86_64.rpm
	
大功告成，在图形界面上查看玩转你的virtualbox吧！