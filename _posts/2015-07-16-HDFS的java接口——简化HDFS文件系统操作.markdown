---
layout:     post
title:      "HDFS的java接口-简化HDFS文件系统操作"
date:       2015-07-16 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	技术 Linux
---

> “这就是我，一个低调的作者。”



今天闲来无事，于是把HDFS的基本操作用java写出简化程序出来给大家一些小小帮助！

	package com.quanttech;

	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;

	/**
	 * @topic HDFS文件操作工具类
	 * @author ZhouJ
	 *
	 */
	public class HdfsUtils {

		/*
		 * 判断HDFS目录是否存在路径path
		 */
		public static boolean isExists(Configuration conf, String path) throws Exception {
			FileSystem fs = FileSystem.get(conf);
			return fs.exists(new Path(path));
		}

		/*
		 * 删除HDFS的一个目录或者文件
		 */
		public static void Delete(Configuration conf, String path) throws Exception {
			FileSystem fs = FileSystem.get(conf);
			fs.delete(new Path(path), true);
		}

		/*
		 * 创建一个HDFS目录
		 */
		public static void Mkdir(Configuration conf, String path) throws Exception {
			FileSystem fs = FileSystem.get(conf);
			if(fs.mkdirs(new Path(path))){
				System.out.println("HDFS目录："+path+"创建成功！");
			}
		}
	}