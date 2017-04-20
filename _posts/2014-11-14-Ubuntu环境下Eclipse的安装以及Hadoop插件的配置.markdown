---
layout:     post
title:      "Ubuntu环境下Eclipse的安装以及Hadoop插件的配置"
date:       2014-11-14 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: Hadoop Ubuntu Eclipse Hadoop插件
grammar_cjkRuby: true
---


## 一、eclipse的安装

在ubuntu桌面模式下，点击任务栏中的ubuntu软件中心，在搜索栏搜索eclipse

![](http://images.cnitblog.com/blog/656602/201411/141631496946120.jpg)

![](http://images.cnitblog.com/blog/656602/201411/141631569914882.jpg)

注意：安装过程需要输入用户密码。

## 二、eclipse的配置

待eclipse安装好以后,在命令行输入whereis eclipse 找到eclipse的安装路径

![](http://images.cnitblog.com/blog/656602/201411/141632066317560.jpg)

在文件目录下找到eclipse中的插件目录

![](http://images.cnitblog.com/blog/656602/201411/141632167107539.jpg)

然后在打开一个文件目录窗口找到hadoop/contrib/eclipse-plugin中的eclipse插件——hadoop-0.20.2-eclipse-plugin.jar

并复制一份

![](http://images.cnitblog.com/blog/656602/201411/141633346005468.jpg)

返回到eclipse目录，右击屏幕中间，选择在终端打开，输入命令：

	sudo chmod 777 * -R
	
![](http://images.cnitblog.com/blog/656602/201411/141633476317949.jpg)

把刚刚复制的插件hadoop-0.20.2-eclipse-plugin.jar，复制到eclipse的插件目录下

![](http://images.cnitblog.com/blog/656602/201411/141633596943502.jpg)

## 三、重新启动eclipse

在eclipse菜单中window下选择preferences

选择hadoop map/reduce 将hadoop的安装目录填写到文本框或者点击browser寻找用户名下hadoop的文件，选中并确定

![](http://images.cnitblog.com/blog/656602/201411/141634146787612.jpg)

在window菜单下选择open perspective，选中map/reduce，并确定

![](http://images.cnitblog.com/blog/656602/201411/141638001632667.png)

在window菜单下选择show view，并选中map/reduse tools中的map/reduce locations，然后确定

![](http://images.cnitblog.com/blog/656602/201411/141638071941115.jpg)

等这些操作做完以后会在eclipse的窗体看到map/reduce locations这个小窗体

![](http://images.cnitblog.com/blog/656602/201411/141638147413420.jpg)

右击map/reduce locations窗体选择new Hadoop location，在弹出的对话框中修改map/reduce master中的port为：9001；DFS master中的port为：9000，在location name中起一个名字（我在这起的名字是：mrchor）

![](http://images.cnitblog.com/blog/656602/201411/141638274287629.jpg)

至此，eclipse安装完毕，且hadoop插件安装完毕。
