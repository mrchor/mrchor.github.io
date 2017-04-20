---
layout:     post
title:      "写一个shell脚本利用wget抓取股票历史数据"
date:       2015-07-13 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 shell 脚本 wget命令 股票
---

> “这就是我，一个低调的作者。”



　　今天，大数据部老大交给我一项任务——抓取股票历史数据。于是乎，我自行在网上找了一下，发现wget真真是一个非常强大的linux下载工具。我已经被深深震撼到了。下面叙述今天的一些过程，还是比较坎坷的。

　　首先，我利用公司现在存在的股票数据，使用hive查询所有的股票代码并导入本地：
  
	hive -e "use stock;select distinct secucode from t_stock_tick_shsz where type='sz';" >> sz_secucode.txt
	hive -e "use stock;select distinct secucode from t_stock_tick_shsz where type='sh';" >> sh_secucode.txt
	
　　PS:上面这一步骤，因为一个小小的问题——开始没有加关键字distinct，结果导致后期抓取数据抓到一大堆重复的股票代码的数据。

　　刚开始想偷懒，想要一句一句地粘贴wget，但是，股票代码太多了，所以还是写脚本吧，shell脚本如下：
  
	#下载上海交易所股票历史记录
	#!/bin/bash                                                           
	  for I in `cat sh_secucode.txt`
		do  
				wget --user-agent="Mozilla/5.0 （Windows; U; Windows NT 6.1; en-US） AppleWebKit/534.16 （KHTML， like Gecko） Chrome/10.0.648.204 Safari/534.16" \
						-nv --tries=5 --timeout=5 -O /home/bigdata/script/zj/sh_history/history_data/$I.csv http://quotes.money.163.com/service/chddata.html?code=0$I&end=20130430                               
				sleep 1s
		done  

	#下载深圳交易所股票历史记录
	#!/bin/bash
			for I in `cat sz_secucode.txt`
			do
				wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.2.3) Gecko/20100401 Firefox/3.6.3 (.NET CLR 3.5.30729)" \
						-nv --tries=5 --timeout=5 -O /home/bigdata/script/zj/sz_history/history_data/$I.csv http://quotes.money.163.com/service/chddata.html?code=1$I&end=20130430
				sleep 1s
			done

　　PS：说一下上面这段代码，为什么在wget有user-agent这个参数？玩过爬虫的同学肯定都知道，当你频繁下载一个网站的东东，这个网站会识别出这是一个爬虫程序，于是就拒绝你下载他家的资源了，所以要设置一个代理，伪装成一个浏览器下载文件，这样被发现的概率就笑了。还有，为什么要加一个sleep？这是因为有可能有的文件比较大，可能在几毫秒之内没有下载完就被挂停了。当然了，我这边的每个文件也就几百K，所以1s也足够了。

　　最后，运行脚本，写这篇文章的时候，脚本还在运行中，希望顺利！O(∩_∩)O