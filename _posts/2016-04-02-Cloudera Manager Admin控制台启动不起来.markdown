---
layout:     post
title:      "Cloudera Manager Admin控制台启动不起来"
date:       2016-04-02 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Cloudera Manager Admin
---

> “这就是我，一个低调的作者。”



这几天都在搞大数据这一块，由于以前自己在弄hadoop等安装的时候特别的费劲，于是乎找到了广大程序员的福音——cloudera manager，但是第一步安装好了以后无法启动，再三思考+百度发现：

通常有以下可能：

service cloudera-scm-server-db 是否启动

service cloudera-scm-server 是否启动

service httpd 是否启动