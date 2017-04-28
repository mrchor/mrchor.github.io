---
layout:     post
title:      "关于Hadoop MapReduce与Apache Spark的二次排序"
date:       2017-4-28 13:35:13
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: MapReduce Spark 二次排序
---

> “这就是我，一个低调的作者。”


我们在做实际业务时，往往会遇到标题所谓的二次排序的问题，例如，我们可以假设以下业务场景：

输入格式：

年 | 月
---|---
2015|09
2014|09
2013|09
2014|03
2016|07
2016|10
2014|11

输出格式：

年|月...
---|---
2013|09
2014|03,09,11
2015|09
2016|07,10

给出上述问题，需要排序年份，还需要在每个年份内部排序月份，本来我们可以这样解决问题：以年份作为key，将不同的key的键值对（<年，月>）送到不同的reduce端，然后在reduce端将每个key对应的value放到一个集合中进行排序输出，然而在实际场景中，我们可能遇到的每个key对应的value值太多，有可能是几万甚至几十万，这样的话，集合就可能会容不下这么多的value，就有可能发生所谓的OOM，因此，我们就需要这样想，即在map端到reduce端的过程中，就讲key对应的内部value排好顺序。这就是所谓的二次排序。

## 一、Hadoop MapReduce篇

在MapReduce中的二次排序中，需要自定义一些类来实现。即我们需要定义一个这样的类：既能将相同年份的数据送到同一个reduce端，又能在送过去的同时是排好序的，于是就有了如下这样一个类了：

{% highlight java %}
    import java.io.DataInput;
    import java.io.DataOutput;
    import java.io.IOException;
    
    import org.apache.hadoop.io.WritableComparable;
    
    /**
     * @author liushaofeng
     */
    public class TextPair implements WritableComparable<TextPair> {//实现WritableComparable接口
    	private String text;
    	private String id;
    
    	public TextPair() {//无参构造器
    
    	}
    	
    	public TextPair(String text, String id) {//新的key值（旧的key+value）的构造器
    		this.text = text;
    		this.id = id;
    	}
    
    	public void setText(String text) {
    		this.text = text;
    	}
    
    	public void setId(String id) {
    		this.id = id;
    	}
    
    	public String getText() {
    		return text;
    	}
    
    	public String getId() {
    		return id;
    	}
    	
    	@Override
    	public void readFields(DataInput in) throws IOException {
    		text = in.readUTF();
    		id = in.readUTF();
    	}
    	
    	@Override
    	public void write(DataOutput out) throws IOException {
    		out.writeUTF(text);
    		out.writeUTF(id);
    	}
    	
    	@Override
    	public int hashCode() {
    		final int prime = 31;
    		int result = 1;
    		result = prime * result + ((id == null) ? 0 : id.hashCode());
    		result = prime * result + ((text == null) ? 0 : text.hashCode());
    		return result;
    	}
    	
    	@Override
    	public boolean equals(Object obj) {
    		if (this == obj)
    			return true;
    		if (obj == null)
    			return false;
    		if (getClass() != obj.getClass())
    			return false;
    		TextPair other = (TextPair) obj;
    		if (id == null) {
    			if (other.id != null)
    				return false;
    		} else if (!id.equals(other.id))
    			return false;
    		if (text == null) {
    			if (other.text != null)
    				return false;
    		} else if (!text.equals(other.text))
    			return false;
    		return true;
    	}
    	
    	@Override
    	public int compareTo(TextPair that) {//在比较方法这边，我们如果我们的新的key中的text不同，则我们就返回这两个不同text的compareTo比较结果；如果相同，那说明是相同的年份，就需要去比较不同的月份，即返回不同月份的compareTo方法比较结果，这样就实现了二次排序了
    		if (!this.text.equals(that.text)) {
    			return this.text.compareTo(that.text);
    		} else {
    			return this.id.compareTo(that.id);
    		}
    	}
    	
    	@Override
    	public String toString() {
    		return "TextPair [text=" + text + ", id=" + id + "]";
    	}
    }
{% endhighlight %}

在定义后二次排序的类后，其实还是不太够的，因为，MapReduce默认的分区和分组的方式不能够识别上面的Writable类，所以我们还需要自定义分区和分组：

继承Partitioner这个类，并覆盖其中的getPartition方法：

{% highlight java %}
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Partitioner;
    
    public class MyPartitioner extends Partitioner<TextPair, Text> {
    	@Override
    	public int getPartition(TextPair key, Text value, int numPartitions) {
    		return Math.abs(key.getText().hashCode() & Integer.MAX_VALUE)
    				% numPartitions;
    	}
    	
    	public static void main(String[] args) {
    		int t1 = 128;
    		int t2 = -128;
    		System.out.println(t1 & Integer.MAX_VALUE);
    		System.out.println(t2 & Integer.MAX_VALUE);
    		System.out.println(t1 & 128);
    		System.out.println(t2 & 128);
    		
    	}
    }
{% endhighlight %}

继承WritableComparator类，覆盖compare方法，即在分组阶段，将相同的key分到一组

{% highlight java %}
    import org.apache.hadoop.io.WritableComparable;
    import org.apache.hadoop.io.WritableComparator;
    
    public class MyGroupingComparator extends WritableComparator {
    	protected GroupingComparator() {
    		super(TextPair.class, true);
    	}
    
    	@Override
    	public int compare(WritableComparable w1, WritableComparable w2) {
    		TextPair ip1 = (TextPair) w1;
    		TextPair ip2 = (TextPair) w2;
    		return ip1.getText().compareTo(ip2.getText());
    	}
    }
{% endhighlight %}

这样，我们通过上述的三个类就可以实现大数据规模场景下的二次排序，而不必担心OOM咯~

## Spark篇

首先，我们先实现一遍简单的二次排序，即先按照年份分组，然后在组内利用一个集合进行排序：

{% highlight scala %}
import org.apache.spark.{SparkConf, SparkContext}

    import scala.collection.mutable.ListBuffer
    
    /**
      * Created by zhoujie on 2017/4/19.
      */
    object PartitionTest {
      val TAB: String = "\t"
    
      def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setAppName("PartitionerTest").setMaster("local")
        val sc: SparkContext = new SparkContext(conf)
        sc.textFile(args(0))
          .map(line => {
            val splited: Array[String] = line.split(TAB, -1)
            (splited(1), splited(0) + TAB + splited(2))
          })
          .groupByKey()
          .sortByKey(true)
          .map(iter => {
            (iter._1, iter._2.toList.sortWith((x, y) => {
              val f: Int = x.split(TAB)(0).compareTo(y.split(TAB)(0))
              if(f>=0){
                true
              }else{
                false
              }
            }))
          })
          .saveAsTextFile(args(1))
      }
    }
{% endhighlight %}

但是，跟MapReduce一样，上述的二次排序还是存在OOM的问题，因此，我们还是需要自定义一些排序规则，形成改进型的二次排序：

首先，也需要自定义一个排序类：

{% highlight scala %}
    /**
      * Created by zhoujie on 2017/4/28.
      */
    class SecondarySortKey(val text: String, val id: String) extends Ordered[SecondarySortKey] with Serializable {
      override def compare(that: SecondarySortKey): Int = {
        if (!this.text.equals(that.text)) {
          this.text.compareTo(that.text)
        } else {
          this.id.compareTo(that.id)
        }
      }
    
      override def <(that: SecondarySortKey): Boolean = {
        if (this.text.compareTo(that.text) < 0) {
          true
        } else {
          false
        }
      }
    
      override def >(that: SecondarySortKey): Boolean = {
        if (this.text.compareTo(that.text) > 0) {
          true
        } else {
          false
        }
      }
    
      override def <=(that: SecondarySortKey): Boolean = {
        if (this.text.equals(that.text) && this.id.compareTo(that.id) < 0) {
          true
        } else {
          false
        }
      }
    
      override def >=(that: SecondarySortKey): Boolean = {
        if (this.text.equals(that.text) && this.id.compareTo(that.id) > 0) {
          true
        } else {
          false
        }
      }
    
      override def toString: String = {
        "[" + this.text + "," + this.id + "]"
      }
    }
{% endhighlight %}

然后，我们在main中这样写：

{% highlight scala %}
    /**
      * Created by zhoujie on 2017/4/19.
      */
    object SecondaryTest {
      val TAB: String = "\t"
    
      def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setAppName("PartitionerTest")
        val sc: SparkContext = new SparkContext(conf)
        sc.textFile(args(0))
          .map(line => {
            val splited: Array[String] = line.split(TAB, -1)
            (new SecondarySortKey(splited(0), splited(1)), splited(2))
          })
          .sortByKey(true)
            .map(tuple => {
              (tuple._1.text, tuple._1.id, tuple._2)
            })
          .saveAsTextFile(args(1))
      }
    }
{% endhighlight %}

通过上述代码，我们就可以类似于MapReduce一样，不需要再集合中进行二次排序了，从而减少了OOM带来的风险。