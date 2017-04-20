---
layout:     post
title:      "Redis在CentOS6.4中的安装"
date:       2016-04-27 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Redis CentOS
---

> “这就是我，一个低调的作者。”



首先，介绍一下Redis数据库。Redis是一种面向“键/值”对数据类型的内存数据库，可以满足我们对海量数据的读写需求。

　　1）redis的键只能是字符串；
　　2）redis的值支持多种数据类型：
  　　　　a：字符串 string

  　　　　b：哈希 hash

  　　　　c：字符串列表 list

  　　　　d：字符串集合 set 不重复，无序

  　　　　e：有序集合sorted set  ，不重复，有序

  　　　　f：HyperLogLog 结构（redis2.8.9版本之后才有,用来做基数统计的算法。）

　　废话不多说了，直接开干！
  
  　一、依赖环境的安装

  由于Redis数据库是C语言开发的，所以，需要在安装Redis数据库的时候，先进行make，make需要C语言的环境，所以先进行C语言编译环境的配置，创建一个脚本写入以下内容并执行：
  
	  #!/usr/bin/bash
			yum -y install cpp
			yum -y install binutils
			yum -y install glibc
			yum -y install glibc-kernheaders
			yum -y install glibc-common
			yum -y install glibc-devel
			yum -y install gcc
			yum -y install make
			
二、编译阶段

1）对下载好的Redis进行解压：点我下载Redis包

	tar -xzvf redis-3.0.7.tar.gz
	
2）进入解压好的Redis目录，编译之，执行：

	make
	
注意：如果此处有错误的话，需要在执行make的时候加个参数：

	make MALLOC=libc
	
3）将Redis所有命令加入环境变量，执行：

	make install
	
4）验证是否成功：

	redis-server --version
	
得到如下结果，说明安装成功！

![](http://images2015.cnblogs.com/blog/656602/201604/656602-20160427190013189-1896465871.png)
