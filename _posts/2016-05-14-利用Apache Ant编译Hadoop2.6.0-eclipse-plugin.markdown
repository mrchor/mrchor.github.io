---
layout:     post
title:      "利用Apache Ant编译Hadoop2.6.0-eclipse-plugin"
date:       2016-05-14 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Apache\bAnt 编译 Hadoop Eclipse 插件
---

> “这就是我，一个低调的作者。”


环境要求：系统不重要，重要的是要有Ant环境，这里不做赘述，自行百度配置去。

1）在github上下载Hadoop-eclipse-plugin-master的zip包，]下载地址](https://codeload.github.com/winghc/hadoop2x-eclipse-plugin/zip/master)。

2）在Hadoop官网下载Hadoop2.6.0，[下载地址](https://www.apache.org/dist/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz)。

3）解压上述俩压缩包。

4）在hadoop2x-eclipse-plugin-master\src\contrib\eclipse-plugin下执行：

	ant jar -Dversion=2.6.0 -Declipse.home=eclipse安装目录 -Dhadoop.home=上述Hadoop解压目录
	我的是：
	ant jar -Dversion=2.6.0 -Declipse.home=D:\ProgramFiles\eclipse -Dhadoop.home=D:\Hadoop\hadoop-2.6.0
	注意：这边的目录不能存在空格！！！
	
5）编译过程中出现错误：

	compile:
		 [echo] contrib: eclipse-plugin
		[javac] D:\Hadoop\hadoop2x-eclipse-plugin-master\src\contrib\eclipse-plugin\build.xml:76: warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds
		[javac] Compiling 45 source files to D:\Hadoop\hadoop2x-eclipse-plugin-master\build\contrib\eclipse-plugin\classes

	BUILD FAILED
	
查资料发现，原来在javac编译的时候默认需要设置一个参数includeAntRuntime="false" 如图：

![](http://images2015.cnblogs.com/blog/656602/201605/656602-20160514225226687-403228390.png)

6）按照这样的步骤，成功了。