---
layout:     post
title:      "Scala中==,eq与equals的区别"
date:       2017-5-31 14:53:16
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: Scala
---

> “这就是我，一个低调的作者。”


## Scala的API
根据API（Scala）的定义：

{% highlight scala %}
	final def ==(arg0: Any): Boolean
	The expression x == that is equivalent to if (x eq null) that eq null else x.equals(that).
	final def eq(arg0: AnyRef): Boolean
	Tests whether the argument (that) is a reference to the receiver object (this).
	def equals(arg0: Any): Boolean
	The equality method for reference types.
{% endhighlight %}

简言之，equals方法是检查值是否相等，而eq方法检查的是引用是否相等。所以如果比较的对象是null那么==调用的是eq，不是null的情况调用的是equals。

看一个简单的例子：
**在java中如果要对两个对象进行值比较，那么必须要实现equals 和hashCode方法。**而在scala中为开发者提供了case class，默认实现了equals 和hashCode方法。

{% highlight scala %}
	scala> case class Bread(brand:String, price:Int)
	defined class Bread

	scala> val b1 = Bread("BreadTalk", 50)
	b1: Bread = Bread(BreadTalk,50)

	scala> val b2 = Bread("BreadTalk", 60)
	b2: Bread = Bread(BreadTalk,60)

	scala> b1 eq b2
	res2: Boolean = false

	scala> b1 equals b2
	res3: Boolean = true
{% endhighlight %}

而对于Array或者Map对象不能简单点使用equals进行值比较，要通过sameElements方法，例如：

{% highlight scala %}
	scala> val a1 = Array("x", "y")
	a1: Array[String] = Array(x, y)

	scala> val a2 = Array("x", "y")
	a2: Array[String] = Array(x, y)

	scala> a1 equals a2
	res4: Boolean = false

	scala> a1 eq a2
	res5: Boolean = false

	scala> a1 sameElements a2
	res6: Boolean = true

	scala> val m1 = Map(1->"x", 2->"y")
	m1: scala.collection.immutable.Map[Int,String] = Map(1 -> x, 2 -> y)

	scala> val m2 = Map(1->"x", 2->"y")
	m2: scala.collection.immutable.Map[Int,String] = Map(1 -> x, 2 -> y)

	scala> m1 sameElements m2
	res7: Boolean = true

	scala> val m3 = Map(1->"x", 2->"z")
	m3: scala.collection.immutable.Map[Int,String] = Map(1 -> x, 2 -> z)

	scala> m1 sameElements m3
	res8: Boolean = false
{% endhighlight %}

如果Array中存的是对象，也是一样的，例如：

{% highlight scala %}
	scala> case class Bread(brand:String, price:Int)
	defined class Bread

	scala> val b1 = Bread("BreadTalk", 50)
	b1: Bread = Bread(BreadTalk,50)

	scala> val b2 = Bread("BreadTalk", 50)
	b2: Bread = Bread(BreadTalk,50)

	scala> val b3 = Bread("BreadTalk", 60)
	b3: Bread = Bread(BreadTalk,60)

	scala> val a1 = Array(b1)
	a1: Array[Bread] = Array(Bread(BreadTalk,50))

	scala> val a2 = Array(b2)
	a2: Array[Bread] = Array(Bread(BreadTalk,50))

	scala> val a3 = Array(b3)
	a3: Array[Bread] = Array(Bread(BreadTalk,60))

	scala> a1 equals a2
	res0: Boolean = false

	scala> a1 sameElements a2
	res1: Boolean = true

	scala> a1 equals a3
	res2: Boolean = false

	scala> a1 sameElements a3
	res3: Boolean = false
{% endhighlight %}

## 与Java比照
首先要记住的是Scala里使用equals方法来比较两个对象，和Java不同。在Java中==操作符用来比较引用的相等性，Scala中==，如果比较对象不是null，则用来比较每个实例相等性的方法，实际上调用的就是equals方法。

方法 | Java | Scala
--|--|--
引用比较（内存地址比较）|==|eq / ==(对象为null时)
对象比较（内容比较）|equals|equals / ==(对象不为null时)

注意：由于Java还不完全是一种面向对象的语言，因此，==不是一个方法，而是一个运算符；而Scala中的==是一个方法，这是本质的区别。




