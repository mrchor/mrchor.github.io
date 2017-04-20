---
layout:     post
title:      "window环境下使用sbt编译spark源码"
date:       2015-07-09 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Spark 编译
---

> “这就是我，一个低调的作者。”



前些天用maven编译打包spark，搞得焦头烂额的，各种错误，层出不穷，想想也是醉了，于是乎，换种方式，使用sbt编译，看看人品如何！

　　首先，从官网spark官网下载spark源码包，解压出来。我这边使用的是1.4.0版本。

　　然后，我们需要把sbt配置好，配置很简单，无非就是SBT_HOME什么的，大家可以参考官网给出的[安装配置手册](http://www.scala-sbt.org/0.13/tutorial/zh-cn/Installing-sbt-on-Windows.html)。
  
在window的命令行模式下进入刚刚解压的spark源码目录下，我们根据官网提示的命令输入：

	sbt -Pyarn -Phadoop-2.3 assembly
	
![](http://images0.cnblogs.com/blog2015/656602/201507/091425462832680.png)
然后等待编译完成。。。

中途报错！！！

![](http://images0.cnblogs.com/blog2015/656602/201507/091426302215591.jpg)

定睛一看，哦！原来是提示没有git命令，于是，我从git官网下载了git，安装并配置好环境变量，这个配置也很简单。

继续编译，心情不好，所以把命令打的更长了：

	sbt -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver assembly

长时间的等待中。。。先去看看hadoop的权威指南。。。

失败，失败，又是失败！

又回到原点，转向了maven。我发现，maven在编译整个spark源码的时候很容易出错，而且出错了找起来也比较麻烦。于是，我决定一个一个小文件夹编译，发现，真的可以诶。现在正在编译小文件夹中的pom.xml。。。

编译完成，讲根目录下的pom.xml修改，删除没必要的module，否则，maven编译测试的时候还是出错，只需要剩下该有的就行。

![](http://images0.cnblogs.com/blog2015/656602/201507/091655417683377.png)