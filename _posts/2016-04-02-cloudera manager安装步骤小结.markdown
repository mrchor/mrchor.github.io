---
layout:     post
title:      "cloudera manager安装步骤小结"
date:       2016-04-02 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 cloudera manager
---

> “这就是我，一个低调的作者。”



1、准备三台虚拟机，系统是centos 7，IP分别是：

　　192.168.254.110　　master

　　192.168.254.111　　slave1

　　192.168.254.112　　slave2

2、如果没有httpd服务的话 需要在master上安装一个httpd：

　　1）　yum install httpd　　#（安装）

　　2）　systemctl start httpd.service　　#（启动）

3、配置ssh免密码登录：

　　具体操作请见我的另一篇文章[《Hadoop2.6.0在CentOS 7中的集群搭建》](/2015/06/06/Hadoop2.6.0在CentOS-7中的集群搭建/index.html)有详述。

4、下载相关文件：

　　地址是：https://archive.cloudera.com/cm5/cm/5/

　　[cloudera-manager-centos7-cm5.6.0_x86_64.tar.gz](https://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.6.0_x86_64.tar.gz)

　　[cloudera-manager-installer.bin](https://archive.cloudera.com/cm5/installer/5.6.0/cloudera-manager-installer.bin)

　　[cm5.6.0-centos7.tar.gz](https://archive.cloudera.com/cm5/repo-as-tarball/5.6.0/cm5.6.0-centos7.tar.gz)

　　[cm5.6.0-centos7.tar.gz.sha1](https://archive.cloudera.com/cm5/repo-as-tarball/5.6.0/cm5.6.0-centos7.tar.gz.sha1)

5、将下载的文件传到master：/root/app/cm中，PS：这个目录需要自己创建

　　进入上述目录，使[cloudera-manager-installer.bin](https://archive.cloudera.com/cm5/installer/5.6.0/cloudera-manager-installer.bin)具有执行权：

	chmod u+x cloudera-manager-installer.bin

　　运行安装命令：

	./cloudera-manager-installer.bin

等待那么几分钟。。。

6：浏览器输入：master：7180即可看到manager的界面，如果打不开，请参考我的另一篇文章[《Cloudera Manager Admin控制台启动不起来》](/2015/04/02/Cloudera-Manager-Admin控制台启动不起来/index.html)

接下来就是配置环节了，我还没有做。。。做好以后分享给大家。