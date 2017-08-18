---
layout:     post
title:      "安装TensorFlow遇到的一系列问题解答"
date:       2017-8-18 16:31:27
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: 技术 shell expect
---

> “这就是我，一个低调的作者。”


其实，在Windows下TensorFlow的安装特别的简单，只需要去[加州大学欧文分校](http://www.lfd.uci.edu/~gohlke/pythonlibs/)下载一个TensorFlow的wheel的安装包就行了。安装也极其简单，只需要用pip install TensorFlow...wheel 就可以了。
但是，当你安装完毕后，使用Python-shell执行：
{% highlight python %}
import tensorflow as tf
{% endhighlight %}

将会得到一系列的错误，如下：
![](/img/2017-08-18/1.png)

这是因为你的Python环境缺乏[matplotlib](http://www.lfd.uci.edu/~gohlke/pythonlibs/#matplotlib)，按照上述安装TensorFlow的方法安装一下。然后它在Python的lib\site-packages下创建一个同名目录，把这个同名目录加到环境编辑就OK了（%PYTHON_HOME%\Lib\site-packages\matplotlib）。

注意，你也可以按照Google给出的[Windows下的TensorFlow安装步骤安装](https://www.tensorflow.org/install/install_windows)进行安装，还有一点就是Windows下，TensorFlow现在只支持Python3.5版本的TensorFlow。


