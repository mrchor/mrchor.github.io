---
layout:     post
title:      "多种语言开发Spark-以WordCount为例"
date:       2017-02-24 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags:	大数据 WordCount Spark
---

> “这就是我，一个低调的作者。”


Spark是目前最火爆的大数据计算框架，有赶超Hadoop MapReduce的趋势。因此，趁着现在还有大多数人不懂得Spark开发的，赶紧好好学习吧，为了使不同的开发人员能够很好的利用Spark，Spark官方提供了不同开发语言的API，本文以大数据经典入门案例WordCount为例，开发多个版本的Spark应用程序，以满足不同的开发人员需求。

一、Scala：

{% highlight scala %}
	val conf: SparkConf = new SparkConf().setMaster("local")
		val sc: SparkContext = new SparkContext(conf)
		sc.textFile("test")
		  .flatMap(line => {
			line.split("\t")
		  })
		  .mapPartitions(iter => {
			val list: List[(String, Int)] = List[(String, Int)]()
			iter.foreach(word => {
			  list.::((word,1))
			})
			list.iterator
		  })
		  .reduceByKey(_ + _)
		  .saveAsTextFile("result")
{% endhighlight %}

二、JDK1.7及以下版本：

{% highlight java %}
	SparkConf conf = new SparkConf().setAppName("JavaSparkTest").setMaster("local");
			JavaSparkContext sc = new JavaSparkContext(conf);
	sc.textFile("test")
			.flatMap(new FlatMapFunction<String, String>() {
				@Override
				public Iterable<String> call(String t) throws Exception {
					return Arrays.asList(t.split("\t"));
				}
			}).mapToPair(new PairFunction<String, String, Integer>() {

				@Override
				public Tuple2<String, Integer> call(String t) throws Exception {
					return new Tuple2<String, Integer>(t, 1);
				}

			}).reduceByKey(new Function2<Integer, Integer, Integer>() {

				@Override
				public Integer call(Integer v1, Integer v2) throws Exception {
					return v1+v2;
				}
			}).saveAsTextFile("result");
{% endhighlight %}

三、JDK1.8：

由于JDK1.8加入了新特性——函数式编程，因此，可以利用JDK1.8的新特性简化Java开发Spark的语句。

{% highlight java %}
	SparkConf conf = new SparkConf().setAppName("JavaSparkTest").setMaster("local");
	JavaSparkContext sc = new JavaSparkContext(conf);
	sc.textFile("test")
			.flatMap(line -> {
				return Arrays.asList(line.split("\t"));
			}).mapToPair(word -> {
				return new Tuple2<String, Integer>(word, 1);
			}).reduceByKey((x, y) -> {
				return x + y;
			}).saveAsTextFile("result");
{% endhighlight %}

是不是觉得比上述的Scala还简洁呢？其实是这样的，Scala中使用了mapPartitions是对map函数的优化，即对每一个RDD的分区进行map操作，这样就减少了对象的创建，从而加速了计算。而Java中，通过我的测试，不能使用mapPartitions方法进行上述优化，只能使用map方法（不知道为啥），这样也可以使用，但是在大数据集面前，其性能就逊色于mapPartitions了。

四、Python：

{% highlight python %}
	from pyspark import SparkContext
	from pyspark import SparkConf as conf
	conf.setAppName("WordCount").setMaster("local")
	sc = SparkContext(conf)

	text_file = sc.textFile("test")\
		.flatMap(lambda line: line.split("\t"))\
		.map(lambda word: (word, 1))\
		.reduceByKey(lambda x, y: x + y)\
		.saveAsTextFile("test")
{% endhighlight %}