---
layout:     post
title:      "Kafka在Centos6.4中的集群搭建"
date:       2016-05-12 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Kafka CentOS 集群
---

> “这就是我，一个低调的作者。”


环境要求：三台装有Centos6.4的虚拟机，需要有java1.7以上的环境，需要ZooKeeper环境。

1）从Kafka官网下载Kafka安装包

[下载Kafka](http://www.cnblogs.com/%E4%B8%8B%E8%BD%BDKafka)

2）解压安装包

	tar -xzf kafka_2.10-0.9.0.1.tgz 
	
3）由于名字太长，改为kafka：

	 mv kafka_2.10-0.9.0.1 kafka
	 
4）进入kafka下面的config目录，修改配置文件server.properties：

	port=9092
	log.dirs=/usr/local/kafka/logs			zookeeper.connect=master:2181,slave1:2181,slave2:2181
	
5）远程复制到其他节点

	scp -r /usr/local/kafka root@slave1:/usr/local/
	scp -r /usr/local/kafka root@slave2:/usr/local/
	
6）在每个节点上后台启动kafka服务

	nohup ./bin/kafka-server-start.sh config/server.properties > /usr/local/kafka/logs/kafka-zk.log 2>&1 &
	
7）启动完成，哦了！