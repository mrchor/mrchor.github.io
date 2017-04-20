---
layout:     post
title:      "eclipse配置hadoop插件"
date:       2015-08-07 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Eclipse Hadoop插件
---

> “这就是我，一个低调的作者。”



1. 版本信息
	eclipse windows 64 bit
	hadoop 2.5.2 64 bit
	hadoop eclipse-plug 2.5.2
 
2. 下载hadoop-2.5.2.tar.gz 
http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.5.2/hadoop-2.5.2.tar.gz 
解压至本地目录
F:/hadoop/hadoop-2.5.2
 
3. eclipse环境配置 
 
3.1.将编译好的hadoop-eclipse-plugin-2.5.2.jar拷贝至eclipse的plugins目录下，然后重启eclipse；

3.2.打开菜单Window--Preference--Hadoop Map/Reduce进行配置，如下图所示：

![](http://images0.cnblogs.com/blog2015/656602/201508/071417138465044.png)

3.3.显示Hadoop连接配置窗口：Window--Show View--Other-MapReduce Tools,如下图所示：

![](http://images0.cnblogs.com/blog2015/656602/201508/071417565807954.png)

3.4.配置连接Hadoop，如下图所示，注意端口号
 
Map/Recudce(V2) Master 端口号是mapred-site.xml 中mapred.job.tracker 属性对应的端口号，如果没配置，默认为9001
 
DFS Master对应的端口为  core-site.xml 中fs.default.name属性对应的端口，如果没有配置，默认为9000

![](http://images0.cnblogs.com/blog2015/656602/201508/071418411595035.png)

3.5 查看是否连接成功，能看到如下信息，则表示连接成功：

![](http://images0.cnblogs.com/blog2015/656602/201508/071419143155540.png)

