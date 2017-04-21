---
layout:     post
title:      "eclipse远程调试Hadoop"
date:       2016-05-31 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Eclipse Maven 
---

> “这就是我，一个低调的作者。”


很多框架都会依赖jdk中的tools.jar，但是maven仓库中却没有.

如在eclipse+maven编写mapreduce代码，就会报Missing artifact　jdk.toos:jdk.toos:jar:1.6

如何解决这个问题呢，只需要在项目的pom.xml 文件中加入以下配置，指定maven去本地寻找 tools.jar

	 	<dependency>
			<groupId>jdk.tools</groupId>
			<artifactId>jdk.tools</artifactId>
			<version>1.6</version>
			<scope>system</scope>
			<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
		</dependency>