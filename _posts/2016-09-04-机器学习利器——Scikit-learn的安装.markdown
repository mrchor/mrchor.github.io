---
layout:     post
title:      "机器学习利器——Scikit-learn的安装"
date:       2016-09-04 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	Python Scikit-Learn 机器学习
---

> “这就是我，一个低调的作者。”


由于笔者最近在进行毕业论文的准备，且毕业论文中需要用到Python版本的机器学习库——scikit-learn。所以最近三天一直在Windows上部署这个框架，终于部署成功了。。。

首先打开加州大学底下一个实验室的网站，下载以下安装包：

1、Numpy+MKL：http://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy

2、Scipy：http://www.lfd.uci.edu/~gohlke/pythonlibs/#scipy

3、scikit-learn：http://www.lfd.uci.edu/~gohlke/pythonlibs/#scikit-learn

前期准备

在部署之前，我们需要在本地环境的Python中有pip模块。这样我们才能安装上述的wheel文件。

一、安装Numpy+MKL：　　

	pip install numpy-1.11.1+mkl-cp27-cp27m-win_amd64.whl
	
二、安装Scipy：

	pip install scipy-0.18.1-cp27-cp27m-win_amd64.whl
	
三、安装scikit-learn：

	pip install scikit_learn-0.17.1-cp27-cp27m-win_amd64.whl
	
这样就大功告成了，前几天一直在看网上的一些安装教程，什么本地VC++编译库啊之类的，都不行，这是我现在能找到最简单的方案了，希望能够节约大家的时间~~~