---
layout:     post
title:      "Scala实现break和continue功能"
date:       2017-5-10 17:32:31
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: Scala
---

> “这就是我，一个低调的作者。”


## 问题
Scala没有像Java一样的break和continue关键字，因此需要使用break或者continue结构的时候该怎么办？

## 解决方案
Scala确实没有上述两个关键字，但是在类scala.util.control.Breaks中提供了类似的功能。注意：在使用的时候，一定要引入这个类哦！

#### break的例子
{% highlight scala %}
    breakable{
        for(i <- 1 to 10){
            println(i)
            if(i > 4) break//跳出循环
        }
    }
{% endhighlight %}

很显然，在i大于4的时候，代码就执行到了break这个方法。此时，一个异常被捕获，breakable“关键字”会捕获这个异常，控制流就被打断了。

请注意：break和breakable实际上不是关键字，而是scala.util.control.Breaks类里面的方法。

#### continue的例子
{% highlight scala %}
    for(i <- 1 to 10){
        breakable{
            if(i == 4) break//跳出循环
            println(i)
        }
    }
{% endhighlight %}

上述代码的意思是当i等于4时，跳出本次循环。具体过程跟break的过程一样。

#### 总结
从这两段代码，我们可以发现，当跳出的条件成立时候，break方法跳出的就是breakable{//代码块}中的花括号，所以只需要记住这一点，你就可以很轻松地玩转Scala的break和continue了。