---
layout:     post
title:      "Ubuntu下的Hadoop搭建"
subtitle:   "\"伪分布式\""
date:       2014-11-13 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:
    - 技术
    - Hadoop
---

> “这就是我，一个低调的作者。”


# 一、必要资源的下载
## 1、Java jdk（jdk-8u25-linux-x64.tar.gz）的下载
具体链接为：
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

## 2、Hadoop（我们在这里选用hadoop0.20.2.tar.gz）的下载

具体链接为：
http://vdisk.weibo.com/s/zNZl3

# 二、软件的安装（将下载好的文件放在home文件夹下）
## 1、Java的安装（解压）

在命令行下输入
{% highlight shell %}

	sudo tar xzvf jdk-8u25-linux-x64.tar.gz
	
{% endhighlight %}
注意：可能会提示你输入用户密码
![](http://images.cnitblog.com/blog/656602/201411/141453589914853.jpg)
解压完毕，Java安装完成

## 2、hadoop的安装（解压）

在命令行输入
{% highlight shell %}

	sudo tar xzvf hadoop0.20.2.tar.gz
	
{% endhighlight %}
注意：可能会提示你输入用户密码

![](http://images.cnitblog.com/blog/656602/201411/141454209446627.jpg)
解压完毕，hadoop0.20.2安装完成

## 3、ssh的安装

在系统联网的情况下，在命令行输入
{% highlight shell %}

	sudo apt-get install ssh
	
{% endhighlight %}
注意：可能提示输入用户密码
![](http://images.cnitblog.com/blog/656602/201411/141454327107921.jpg)
## 4、rsync的安装

在系统联网的情况下，在命令行输入
{% highlight shell %}

	sudo apt-get install rsync
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141454466781088.jpg)
# 三、环境的配置
## 1、ssh的配置

ssh需要配置成为免密码登录状态

在命令行输入两句话：
{% highlight shell %}

	ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
	cat ~/.ssh/id_rsa.pub >>~/.ssh/authorized_keys
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141455021319742.jpg)
配置完毕，验证ssh是否需要密码，在命令行输入：
{% highlight shell %}

	ssh localhost
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141455193973685.jpg)
以下操作在hadoop-0.20.2文件夹中的conf下进行，在命令行输入 cd Hadoop-0.20.2/conf

如果不能修改下面的文件的话，在home目录下命令行输入：
{% highlight shell %}

	sudo chmod 777 * -R
	
{% endhighlight %}
以下的文件修改还可以用vi修改不熟悉vi的同学，请先学一下vi操作

以下的文件修改也可以在文件目录直接用gedit打开修改
## 2、修改hadoop-env.sh中的配置

先找到Java安装目录
![](http://images.cnitblog.com/blog/656602/201411/141455374915812.jpg)
将JAVA_HOME改成你安装Java JDK的绝对路径
{% highlight shell %}

	gedit hadoop-env.sh
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141455560063769.jpg)
## 3、修改hadoop核心配置文件core-site.xml
{% highlight shell %}

	gedit core-site.xml
	
{% endhighlight %}
{% highlight xml %}

	<configuration>
		<property>
			<name>fs.default.name</name>
			<value>hdfs://localhost:9000</value>
		</property>
	</configuration>
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141456140066867.jpg)
## 4、修改hadoop中HDFS的配置，修改replication
{% highlight shell %}

	gedit hdfs-site.xml
	
{% endhighlight %}
{% highlight xml %}	

	<configuration>
		<property>
           <name>dfs.data.dir</name>
           <value>/home/mrchor/hadoop-0.20.2/data</value>
		</property>
		<property>
           <name>dfs.replication</name>
           <value>1</value>
		</property>
	</configuration>
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141500502728402.jpg)
## 5、修改hadoop中MapReduce的配置文件，配置的是JobTracker的地址和端口
{% highlight shell %}

	gedit mapred-site.xml
	
{% endhighlight %}
{% highlight shell %}

	<configuration>
		<property>
           <name>mapred.job.tracker</name>
           <value>localhost:9001</value>
		</property>
	</configuration>
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141501126006289.jpg)
# 四、hadoop的启动
以下操作在hadoop-0.20.2文件夹下进行，在命令行输入：cd hadoop-0.20.2

## 1、格式化hadoop中的文件系统HDFS
{% highlight shell %}

	bin/hadoop namenode –format
	
{% endhighlight %}
![](http://images.cnitblog.com/blog/656602/201411/141457218199518.jpg)
## 2、启动hadoop环境
{% highlight shell %}

	bin/start-all.sh
	
{% endhighlight %}	
![](http://images.cnitblog.com/blog/656602/201411/141457331479213.jpg)
# 五、验证
需要在浏览器中输入localhost：50030和localhost：50070验证hadoop是否安装完成
![](http://images.cnitblog.com/blog/656602/201411/141457485065138.jpg)
![](http://images.cnitblog.com/blog/656602/201411/141458016478120.jpg)
至此，hadoop的环境搭建完毕。


2017年4月20日，经过几经折腾，我的个人博客站，终于开通了，是基于GitHub的免费空间，全部都是静态页面。

以前是在博客园写技术，名字叫[低调才是王道-博客园](www.cnblogs.com/cstzhou)，虽然说博客园在程序员界还是有一定影响的，但是编辑器貌似不太给力，有时候写的博客格式过段时间都乱了，所以，再过些天，准备把那边的文章看看能有什么方法转过来，不要浪费以前的东西~

