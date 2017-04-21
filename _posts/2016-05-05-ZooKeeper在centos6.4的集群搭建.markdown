---
layout:     post
title:      "ZooKeeper在centos6.4的集群搭建"
date:       2016-05-05 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	ZooKeeper centos 集群
---

> “这就是我，一个低调的作者。”


首先给一个小tips，在搭建zookeeper之前，需要配置好java环境，请参看我的另一篇文章[《Jdk1.8在CentOS7中的安装与配置》](/2015/06/05/Jdk1.8在CentOS7中的安装与配置/index.html)，还需要免密码登录，请参看我的另一篇文章[《Hadoop2.6.0在CentOS 7中的集群搭建》](/2015/06/06/Hadoop2.6.0在CentOS-7中的集群搭建/index.html)。

集群配置信息：

	  server.0——192.168.10.110  master
	  server.1——192.168.10.120  slave1
	  server.2——192.168.10.130  slave2

下面开干！

一 、下载zookeeper的tar包

　　在Linux命令行执行：
  
	wget  http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz

二、待下载完成后

解压tar包：

	tar -xzvf  zookeeper-3.4.8.tar.gz 
	
修改解压后的zookeeper名字：

	mv zookeeper-3.4.8 zookeeper
	
三、修改配置文件

	cd zookeeper/conf
	cp zoo_sample.cfg zoo.cfg
	vi zoo.cfg

1）指定zookeeper数据存储文件目录，注意：这个目录需要自行创建

	dataDir=/usr/local/zookeeper/data

2）指定集群中zookeeper节点的编号和端口

	server.0=master:2888:3888
	server.1=slave1:2888:3888
	server.2=slave2:2888:3888
	
3）每一个节点拷贝一份上述修改的zookeeper文件夹

	scp -r zookeeper/  slave1:/usr/local/
	scp -r zookeeper/  slave2:/usr/local/
	
4）分别在各个节点的zookeeper/data中创建一个文件myid，内容分别是 0、1、2

四、启动集群，分别在集群的节点启动zookeeper服务

在zookeeper目录下执行：

	./bin/zkServer.sh start

五、集群搭建完毕

至此，ZooKeeper服务集群搭建完毕！