---
layout:     post
title:      "网络爬虫框架Heritrix中Modules的各项说明"
date:       2015-01-22 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Heritrix
---

> “这就是我，一个低调的学者。”



　　1）Select Crawl Scope：Crawl Scope 用于配置当前应该在什么范围内抓取网页链接。例如选择 BroadScope 则表示当前的抓取范围不受限制，选择 HostScope 则表示抓取的范围在当前的 Host 范围内。在这里我们选择 org.archive.crawler.scope.BroadScope，并单击右边的 Change 按钮保存设置状态。

　　2）Select URI Frontier：Frontier 是一个 URL 的处理器，它决定下一个被处理的 URL 是什么。同时，它还会将经由处理器链解析出来的 URL 加入到等待处理的队列中去。这里我们使用默认值。

　　3）Select Pre Processors：这个队列的处理器是用来对抓取时的一些先决条件进行判断。比如判断 robot.txt 信息等，它是整个处理器链的入口。这里我们使用默认值。

　　4）Select Fetchers：这个参数用于解析网络传输协议，比如解析 DNS、HTTP 或 FTP 等。这里我们使用默认值。

　　5）Select Extractors：主要是用于解析当前服务器返回的内容，取出页面中的 URL，等待下次继续抓取。这里我们使用默认值。

　　6）Select Writers：它主要用于设定将所抓取到的信息以何种形式写入磁盘。一种是采用压缩的方式（Arc），还有一种是镜像方式（Mirror）。这里我们选择简单直观的镜像方式：org.archive.crawler.writer.MirrorWriterProcessor。

　　7）Select Post Processors：这个参数主要用于抓取解析过程结束后的扫尾工作，比如将 Extrator 解析出来的 URL 有条件地加入到待处理的队列中去。这里我们使用默认值。
