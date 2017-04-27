---
layout:     post
title:      "开发Apache Storm遇到的问题"
date:       2017-04-27 12:00:00
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: Apache\bStorm 开发问题
---

> “这就是我，一个低调的作者。”


## 一、包冲突
在开发storm时，由于会引入很多依赖包，因此，不免就会造成包冲突的问题，一般的问题都是日志框架的依赖重复，如下：

{% highlight shell %}
    SLF4J: Detected both log4j-over-slf4j.jar AND slf4j-log4j12.jar on the class path, preempting StackOverflowError. 
    SLF4J: See also http://www.slf4j.org/codes.html#log4jDelegationLoop for more details.
{% endhighlight %}

这个是非常常见，但是很头疼的问题，我个人百度了好久，利用mvn dependency:tree 查看到了重复的包，也是坑了好久，最后索性不去留一个日志框架了，直接在Eclipse把所有的日志框架都剪掉，然后我自己添加了一个日志框架：

{% highlight xml %}
    <dependency>
        <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
        <version>1.7.7</version>
    </dependency>
{% endhighlight %}

然后，再次提交storm的Topology，又发现了新的问题，下一个问题。
## 二、Log4J序列化问题
发布topologies 时，出现不能序列化log4j.Logger 的异常：

{% highlight shell %}
    Exception in thread "main" java.lang.IllegalStateException: Bolt 'BottomMenuStatBolt' contains a non-serializable field of type org.apache.log4j.Logger, which was instantiated prior to topology creation. org.apache.log4j.Logger should be instantiated within the prepare method of 'BottomMenuStatBolt at the earliest.
{% endhighlight %}

在这里，我们有两种解决方案：
###### 1）Log4J框架换成Slf4J框架（网上说的）；但是貌似不是很容易解决问题；
###### 2）把声明为Logger的变量忽略序列化即可：

{% highlight java %}
    transient Logger logger = Logger.getLogger(BottomMenuStatBolt.class);
{% endhighlight %}

这种方式还是很好解决问题的。

## 三、没有定义输出列

{% highlight shell %}
    Component: [BottomMenuStatBolt] subscribes from non-existent stream: [BottomMenuClickStat] of component [BottomMenuFilterBolt]
{% endhighlight %}

检查Spout和Bolt代码中的declareOutputFields方法
declare的Field数量 等于 collector.emit数量

## 四、没有发现类的异常

{% highlight shell %}
    java.lang.NoClassDefFoundError: kafka/javaapi/consumer/SimpleConsumer at storm.kafka.DynamicPartitionConnections.register(DynamicPartitionConnections.java:33) at storm.kafka.PartitionManager.<init>(
{% endhighlight %}

由于我在maven引入了本地的kafka.jar，因此，在打包的时候无法将本地包打入到maven的jar里面（PS：scope为system时，与provided相类似，不加入maven的带有依赖的jar中），因此，我采用了另外一种解决方案，即将本地包安装到我的maven本地库中：

{% highlight shell %}
    >mvn install:install-file -Dfile=./kafka-0.7.1.jar -DgroupId=org.apache.kafka -DartifactId=kafka -Dversion=0.7.1 -Dpackaging=jar
{% endhighlight %}

然后，我在pom中引用的时候就可以将scope设置为compile，然后就可以打包到maven的jar了。

PS：上述这个问题的原因，也有可能是因为引入的jar包版本的问题导致，需要找到对应版本的jar，如果在maven上没有的话，还需要自行下载，然后按照上面给出的步骤安装到本地maven仓库~

## 五、看起来代码都很正常，但是bolt无法emit数据到下一个bolt

这个可能是因为你写的collector变量名不对，我就遇到了这样的坑，原本我定义了_collector，prepare方法中这么写：

{% highlight java %}
    @Override
    public void prepare(Map stormConf, TopologyContext context,
    		OutputCollector collector) {
    	this._collector = collector;
    }
{% endhighlight %}

但是，请注意，这里由于Clojure这门语言我不是很了解，我认为是可能带有_的变量，加上this会有问题，因此，我重新定义了collector，上述代码也变成了：

{% highlight java %}
    @Override
    public void prepare(Map stormConf, TopologyContext context,
    		OutputCollector collector) {
    	this.collector = collector;
    }
{% endhighlight %}
