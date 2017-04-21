---
layout:     post
title:      "Hadoop MapReduce编程创建maven项目时所用到的pom依赖"
date:       2016-05-07 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Hadoop MapReduce maven pom 依赖
---

> “这就是我，一个低调的作者。”


	<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>3.8.1</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>org.apache.hadoop</groupId>
				<artifactId>hadoop-common</artifactId>
				<version>2.6.0</version>
			</dependency>
			<dependency>
				<groupId>org.apache.hadoop</groupId>
				<artifactId>hadoop-client</artifactId>
				<version>2.6.0</version>
			</dependency>
			<dependency>
				<groupId>org.apache.hadoop</groupId>
				<artifactId>hadoop-hdfs</artifactId>
				<version>2.6.0</version>
			</dependency>
			<dependency>
				<groupId>jdk.tools</groupId>
				<artifactId>jdk.tools</artifactId>
				<version>1.7</version>
				<scope>system</scope>
				<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
			</dependency>
