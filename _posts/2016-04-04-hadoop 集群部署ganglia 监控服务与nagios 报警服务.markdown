---
layout:     post
title:      "hadoop 集群部署ganglia 监控服务与nagios 报警服务"
date:       2016-04-04 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Hadoop Ganglia Nagios
---

> “这就是我，一个低调的做者。”



1. 部署ganglia 服务
 
ganglia 涉及到的组件:
 
1)数据监测节点（gmond）：这个部件装在需要监测的节点上，用于收集本节点的运行情况，并将这些统计信息传送到gmetad，Ubuntu系统中的ganglia-monitor包可以安装；
2)数据收集节点（gmetad、gweb）：这个部件用于收集gmond发送的数据，并通过web部件将其显示处理，可以通过ganglia-webfrontend包完成安装；
3)web界面：这个就是用于将gmetad整理生成的xml数据以网页形式显示出来的部件，已经包含在了ganglia-webfrontend包.

gmetad负责收集各个节点的监测数据，在集群中只需要选择一台节点进行安装即可，然后可以通过这台节点获取监测结果；gmond负责监测各个节点，所以在集群中的每个节点上都需要安装gmond。

1.1 监控主节点(信息收集与展示节点)部署 192.168.2.237

	sudo apt-get install ganglia-monitor
	sudo apt-get install ganglia-webfrontend
	
本节点作为主节点，用于收集并展示其他节点的信息。
 
复制 Ganglia webfrontend Apache 配置

	sudo cp /etc/ganglia-webfrontend/apache.conf /etc/apache2/sites-enabled/ganglia.conf

在进行Ganglia集群配置之前，首先要搞清楚单播和组播。

1)单播：可以跨网段传播，只将信息发送给指定的机器。要配置成为单播你应该指定一个（或者多个）接受的主机。
2)组播：在机器所处的网段中发送广播，发送给位于同一网段的所有机器。如果你正在使用组播传输，那么你没必要改变任何东西，因为这是Ganglia 包安装默认的。唯一要做的就是把gmetad指向一个或几个运行着gmo nd的主机。没有必要列出每一个单个主机，因为gmo nd被设置为接受模式时会包含所有主机的列表以及整个集群的统计信息。

本集群使用了单播模式，在每个机器上都要配置/etc/ganglia/gmond.conf  

	globals {
	daemonize = yes
	setuid = yes
	user = root /*运行Ganglia的用户*/
	debug_level = 0
	max_udp_msg_len = 1472
	mute = no
	deaf = no
	host_dmax = 120 /*secs */
	cleanup_threshold = 300 /*secs */
	gexec = no
	send_metadata_interval = 10/*发送数据的时间间隔*/
	}

	cluster {
	name = "hadoop" /*集群名称*/
	owner = "root" /*运行Ganglia的用户*/
	latlong = "unspecified"
	url = "unspecified"
	}

	udp_send_channel {
	# mcast_join = 192.168.52.105 /*注释掉组播*/
	host = 192.168.2.237 /*发送给安装gmetad的机器*/
	port = 8649
	ttl = 1
	}

	udp_recv_channel { #mcast_join = 239.2.11.71 port = 8649 #bind = 239.2.11.71 }
	
配置 /etc/ganglia/gmetad.conf,该配置在仅在主节点配置。
data_source "hadoop" 192.168.2.237:8649,192.168.2.201:8649,192.168.2.202:8649,192.168.2.203:8649,192.168.2.204:8649,192.168.2.205:8649,192.168.2.206:8649,192.168.2.207:8649,192.168.2.208:8649,192.168.2.209:8649,192.168.2.210:8649,192.168.2.211:8649,192.168.2.212:8649,192.168.2.213:8649,192.168.2.214:8649,192.168.2.215:8649,192.168.2.216:8649
 
注意，此处的hadoop 一定要对应 /etc/ganglia/gmond.conf 里的cluster 的name="hadoop"，并且不同的cluster 一定要对应不同的端口

重启服务

	sudo /etc/init.d/ganglia-monitor restart
	sudo /etc/init.d/gmetad restart
	sudo /etc/init.d/apache2 restart
	
1.2 从节点部署

从节点只安装gmond即可,配置同上面 /etc/ganglia/gmond.conf  

	sudo apt-get install ganglia-monitor
	
重启服务
 
	sudo /etc/init.d/ganglia-monitor restart
	
2 监控Hadoop集群
 
编辑文件 hadoop-2.5.2/etc/hadoop/hadoop-metrics2.properties

	#
	# Licensed to the Apache Software Foundation (ASF) under one or more
	# contributor license agreements. See the NOTICE file distributed with
	# this work for additional information regarding copyright ownership.
	# The ASF licenses this file to You under the Apache License, Version 2.0
	# (the "License"); you may not use this file except in compliance with
	# the License. You may obtain a copy of the License at
	#
	# http://www.apache.org/licenses/LICENSE-2.0
	#
	# Unless required by applicable law or agreed to in writing, software
	# distributed under the License is distributed on an "AS IS" BASIS,
	# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	# See the License for the specific language governing permissions and
	# limitations under the License.
	#

	# syntax: [prefix].[source|sink].[instance].[options]
	# See javadoc of package-info.java for org.apache.hadoop.metrics2 for details

	#*.sink.file.class=org.apache.hadoop.metrics2.sink.FileSink
	# default sampling period, in seconds
	#*.period=10

	# The namenode-metrics.out will contain metrics from all context
	#namenode.sink.file.filename=namenode-metrics.out
	# Specifying a special sampling period for namenode:
	#namenode.sink.*.period=8

	#datanode.sink.file.filename=datanode-metrics.out

	# the following example split metrics of different
	# context to different sinks (in this case files)
	#jobtracker.sink.file_jvm.context=jvm
	#jobtracker.sink.file_jvm.filename=jobtracker-jvm-metrics.out
	#jobtracker.sink.file_mapred.context=mapred
	#jobtracker.sink.file_mapred.filename=jobtracker-mapred-metrics.out

	#tasktracker.sink.file.filename=tasktracker-metrics.out

	#maptask.sink.file.filename=maptask-metrics.out

	#reducetask.sink.file.filename=reducetask-metrics.out
	*.sink.ganglia.class=org.apache.hadoop.metrics2.sink.ganglia.GangliaSink31

	*.sink.ganglia.period=10

	# default for supportsparse is false
	*.sink.ganglia.supportsparse=true

	*.sink.ganglia.slope=jvm.metrics.gcCount=zero,jvm.metrics.memHeapUsedM=both
	*.sink.ganglia.dmax=jvm.metrics.threadsBlocked=70,jvm.metrics.memHeapUsedM=40

	namenode.sink.ganglia.servers=192.168.2.237:8649

	datanode.sink.ganglia.servers=192.168.2.237:8649

	jobtracker.sink.ganglia.servers=192.168.2.237:8649

	tasktracker.sink.ganglia.servers=192.168.2.237:8649

	maptask.sink.ganglia.servers=192.168.2.237:8649

	reducetask.sink.ganglia.servers=192.168.2.237:8649
	
重启各项服务。