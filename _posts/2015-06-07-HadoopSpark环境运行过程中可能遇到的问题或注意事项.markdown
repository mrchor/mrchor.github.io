---
layout:     post
title:      "Hadoop/Spark环境运行过程中可能遇到的问题或注意事项"
date:       2015-06-07 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Hadoop Spark 大数据
---

> “这就是我，一个低调的作者。”



## 1、集群启动的时候，从节点的datanode没有启动

问题原因：从节点的tmp/data下的配置文件中的clusterID与主节点的tmp/data下的配置文件中的clusterID不一致，导致集群启动时，hadoop会杀死从节点的datanode进程。

解决方案：

a)　　将集群关闭;

b)　　删除你在hadoop配置中设置的tmp下的data和name中的内容（每一个节点都要做这个操作）

c)　　重新格式化一次hdfs

d)　　重启集群，问题解决

## 2、集群启动时，jps显示所有的hadoop进程都已经存在，但是宿主机的浏览器打不开监控页面

问题原因：集群中的所有节点的防火墙没有被禁用，导致宿主机无法访问监控界面。

解决方案：

a)　　将防火墙禁用（每一个节点都要做这个操作）

	centos 7：
	systemctl stop firewalld.service #停止
	systemctl disable firewalld.service #禁用

	之前的版本：　　　　　　　　　　
	service iptables stop #停止
	chkconfig iptables off #禁用

b)　　问题解决。
## 3、启动sparkshell的时候出现如下错误：
	
	Call From master to master:8020 failed on connection exception: java.net.ConnectException: Connection refused

问题原因：端口设置错误或者集群未启动导致通信失败
解决方案：
　　　a)　　先jps查看是否集群启动，如果启动则非此原因
　　　b)　　查看hdfs配置时候端口是8020
　　　c)　　hdfs默认端口为9000
   
## 4、提交任务到集群的时候报错：
	ERROR SparkDeploySchedulerBackend: Application has been killed. Reason: All masters are unresponsive!

解决过程：
　　a) 先前我以为是scala版本不对，因为官网上spark默认所支持的scala版本是scala2.10，想要支持scala2.11.需要自行编译。而我用的时scala2.11，所以我把集群中scala版本全部换成2.10版本。但是问题未得到解决。
　　b) 上网看到有人遇到相同的问题，说是spark提交任务的时候（如果通信工具没有改变的话，kafka另论），默认使用spark自带的通信工具akka，但是akka只能够识别IP主机映射的hostname，而无法识别IP地址，所以我把集群中的每个节点的spark配置文件中的spark_master_ip修改为hostname（master主节点名字），集群重启后，问题得到解决。
  
  ## 5、在Spark集群提交任务后报错：
  	You need to build Spark before running this program.
	Initial job has not accepted any resources; check your cluster UI to ensure
通过查看日志发现，主节点的配置文件spark-evn.sh无故丢失（具体原因不详，可能是我在操作的时候在UI界面kill了一个任务导致），于是把其他节点的spark-evn.sh复制到主节点，集群服务全部关闭，主节点重启，服务重启，问题解决。
## 6、Spark的Application（print一个结果）在提交到yarn的时候，成功执行完成，但是没有打印结果：
Spark提交任务到yarn的时候有两种模式：yarn-client和yarn-cluster，yarn-client适合于日常生产，而yarn-client更适合于交互，可以作为测试使用。详细介绍请参看：[《Spark:Yarn-cluster和Yarn-client区别与联系》](https://www.iteblog.com/archives/1223.html)
所以，刚刚提交任务的时候我采用的是cluster模式，故没有打印结果，换成client模式就可以了。

## 7、Spark提交任务后，Application运行成功，但是在SparkUI没有显示Application：
原因：Spark的缺省配置spark-default.conf没有打开 
解决方案：去掉spark-default.conf.template最后的 .template，重启集群，问题解决！
PS： 其实1.4版本不打开这个spark-default.conf.template，spark默认在UI不可以显示的，这是因为你设置的主节点主机名不是master，而spark默认显示是master，所以UI无法正确显示Application。 另外，spark-submit提交任务到其他集群，在SparkUI中也是无法显示的，只能在对应的集群管理界面找到，一定要注意这一点！
## 8、Spark On Yarn运行时，设置历史记录：spark.history.fs.logDirectory要与spark.eventLog.dir指向同一目录，否则无法正确显示历史日志！
## 9、Spark On Yarn提交任务模板：spark-submit --master yarn-cluster --deploy-mode cluster --class com.quanttech.ASL.MoviesRecommond 
	hdfs://192.168.2.201:54310/user/bigdata/script/zj/ScalaTestRecommond.jar
如有其他问题，我会后续更新！O(∩_∩)O