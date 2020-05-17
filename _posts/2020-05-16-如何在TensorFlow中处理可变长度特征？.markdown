---
layout:     post
title:      "如何在TensorFlow中处理可变长度特征？"
date:       2020-5-16 20:15:09
author:     "Mrchor"
header-img: "img/post-bg-2015.jpg"
catalog:	true
tags: TensorFlow 可变长度特征处理
---

> “这就是我，一个低调的作者。”


## 前言
在一些算法应用的场景中，我们经常会遇到一些带权可变长度特征需要处理的情况。那么，什么是带权可变长度特征呢？其实这个定义就是说，有一个特征，但是这个特征是一系列子特征的集合。举个例子，例如在电商场景下，用户在浏览网页的时候会产生一些该用户对于的分类，那么此时不同的用户对应的喜欢的东西是不一样的，且他们的浏览的分类个数也是不尽相同的。假设用户A浏览了，鞋子、衣服、袜子，而用户B浏览了手机、笔记本，这时我们如果把用户浏览的分类轨迹作为特征传入到模型进行训练或线上推理，我们该怎么做呢？

## 简单截断
这个思路是参考了自然语言处理的（Natural Language Processing）解决方案，顾名思义，就是说对用户的浏览分类轨迹，如果太长就进行截断成固定长度，如果太短就进行空白填补（padding）成固定长度。
具体的代码，我们不做详述，网上的例子很多，大家可以照猫画虎。我们在此说明一下这种处理方式的缺点。即在对特征数据进行截断的时候，无意中，我们已经把用户的一些特征信息进行了“弱化”，也就是中，在进行截断的时候，我们人为地对用户的特征进行了表达能力的减弱，这其实是不利于我们模型对该用户特征信息挖掘的。因此，引出了我们今天的主角，也就是另外一种处理方式——可变长度特征处理。


## 可变长度特征处理
可变长度特征处理，在不丢失用户可变长度特征信息的同时，对用户的可变长度特征进行信息表达，具体有两种结果方式：

#### 处理成multi-hot特征
直接给出例子：
假设我们现在的电商上的分类集合为：{未知分类、鞋子、手机、衣服、袜子、笔记本}
用户A的分类浏览特征是：鞋子、衣服、袜子
那么，我们经过TensorFlow处理后得到的应该是：0,1,0,1,1,0
同样用户B的分类浏览特征是：手机、笔记本
那么，经过TensorFlow处理后得到的是：0,0,1,0,0,1
在代码中如何实现呢？看下面：
思路：
首先，我们利用TensorFlow自带的index_table对象 ，初始化构建一个分类集合的索引表，这个索引表可以返回每个分类对应的索引位置ID：
``` python
CATE_SET=[‘未知分类’,'鞋子','手机','衣服','袜子','笔记本']
cate_table = tf.contrib.lookup.index_table_from_tensor(
        mapping=CATE_SET, num_oov_buckets=1, default_value=0
)
```
然后，我们对用户的原始分类浏览特征进行切割处理：
``` python
user_A_cate_browse='鞋子,衣服,袜子'
#此处得到的是一个sparse_tensor
user_A_cates = tf.string_split([user_A_cate_browse], sep, skip_empty=False)
```
第三步，利用索引表返回用户对应的分类特征的索引id，并按照索引表大小建立one-hot编码：
```python
user_A_cates_one_hot = tf.one_hot(cate_table.lookup(user_A_cates.values), len(CATE_SET))
```
此时用户的one-hot编码对应的是他浏览的每一个的分类对应的one-hot编码二维tensor，所以我们需要进行一次reduce，将其转换为multi-hot编码特征：
```python
user_A_cate3_multi_hot = tf.reduce_sum(user_A_cates_one_hot,0)
```
这样，我们就做出了该用户对应的浏览分类的multi-hot编码特征。
#### 处理成embedding特征
上述的multi-hot所做出来的编码特征对于分类这种维度较小的集合来说，不会太大增加我们最终整体的特征维度，但是如果上述处理应用到商品这个维度，那么势必会对最终特征维度造成极大的维度压力，也就是所谓的“维度爆炸”，此时我们可以利用另外一个处理办法，也就是进行embedding处理。具体如下：
第一步，还是要先进行index_table的初始化，这边直接给出代码：
```python
CATE_SET=[‘未知分类’,'鞋子','手机','衣服','袜子','笔记本']
cate_table = tf.contrib.lookup.index_table_from_tensor(
        mapping=CATE_SET, num_oov_buckets=1, default_value=0
)
```
第二步，跟上面一样，也是要对用户的原始分类浏览特征进行切割处理：
```python
user_A_cate_browse='鞋子,衣服,袜子'
#此处得到的是一个sparse_tensor
user_A_cates = tf.string_split([user_A_cate_browse], sep, skip_empty=False)
```
第三步开始不一样了，利用user_A_cates的索引tensor和经过索引表处理的user_A_cates的值tensor对应的索引值，构造一个SparseTensor：
```python
user_A_cates_sparse = tf.SparseTensor(
      indices=user_A_cates.indices,
      values=table.lookup(user_A_cates.values),
      dense_shape=user_A_cates.dense_shape)
```
第四步，构造一个embedding参数矩阵，并进行embedding_lookup操作，即可得到该用户的分类浏览的embedding特征:
```python
embedding_params = tf.Variable(tf.truncated_normal([len(TAG_SET) + NUM_OOV, TAG_EMBEDDING_DIM]))
user_A_cates_emb = tf.nn.embedding_lookup_sparse(embedding_params, sp_ids=user_A_cates_sparse, sp_weights=None)
```
## 带权可变长度特征处理
经过上述处理，可以说我们已经实现了对可变长度特征不做信息损耗的特征处理，但是同学们不知道有没有考虑一个问题：用户在浏览对应商品分类的时候，其实潜在存在着一种浏览前后分类偏好重要性的问题。例如用户C，在电商网站上浏览了10次手机，浏览了1次衣服，那么，我们完全可以认为用户C其实是更想要去买手机，而对于购买衣服的意愿貌似不是那么强烈。因此，我们就需要将用户浏览的次数加入到特征处理，以表征用户对于分类的喜好程度，这就牵扯出了带权可变长度特征如何处理的问题。我们在这一部分跟上一小节一样也是分两种处理方式，multi-hot和embedding。
假设我们的用户C的带权浏览分类特征原生形式为：
```python
user_C_cate_browse='手机,衣服|9,1'
```
#### 带权multi-hot处理
第一步，跟上面一样，进行索引表初始化：
```python
#索引表初始化
CATE_SET=['未知分类','鞋子','手机','衣服','袜子','笔记本']
cate_table = tf.contrib.lookup.index_table_from_tensor(
        mapping=CATE_SET, num_oov_buckets=1, default_value=0
)
```
第二步，对用户C的带权浏览分类特征进行两步切割，分别求出用户的浏览分类，以及各浏览分类对应的浏览次数：
```python
user_C_cate_browse_split = tf.string_split([user_C_cate_browse], '|', skip_empty=False)
# 浏览分类进行二次切割
user_C_cates = tf.string_split([user_C_cate_browse_split.values[0]], ',', skip_empty=False)

# 特征权重进行二次切割，并进行字符串转浮点型
user_C_cates_wgts = tf.string_to_number(tf.string_split([user_C_cate_browse_split.values[1]], ',').values, out_type=tf.float32)
```
第三步，我们这边利用了一个非常巧妙的技巧
用户C的浏览分类的权重，我们先转成了一个权重矩阵:
``` text
[9.0, 1.0]
```
而用户C的浏览分类经过索引表处理，得到了对应的分类索引值，然后在经过one-hot编码，形成一个one-hot编码分类矩阵，即：
``` text
[
    [0,0,1,0,0,0],
    [0,0,0,1,0,0]
]
```
此时，我们利用两个矩阵的乘积，就可以得到我们对应的带权重的multi-hot编码了，即如下处理:
``` python
tf.matmul(
    tf.reshape(user_C_cates_wgts, [1, -1]),
    tf.one_hot(cate_table.lookup(user_C_cates.values), vocabulary_size)
)
```
#### 带权embedding特征处理
前面的一二步骤一致，不在此赘述。我们直接从第三步开始：
同学们是否以及发现，我们在第二小节中的embedding处理中在embedding_lookup处理时，有一个sp_weights权重，那么也就是说我们可以将分类权重作为sp_weights的参数传入，此时就简单了：
``` python
#构造一个embedding参数矩阵，并进行embedding_lookup操作，即可得到该用户的分类浏览的embedding特征
embedding_params = tf.Variable(tf.truncated_normal([len(TAG_SET) + NUM_OOV, TAG_EMBEDDING_DIM]))
user_A_cates_emb = tf.nn.embedding_lookup_sparse(embedding_params, sp_ids=user_A_cates_sparse, sp_weights= tf.reshape(user_C_cates_wgts, [1, -1]))
```
经过处理，我们就得到了用户C带权的分类浏览的embedding特征。
## 总结
上述介绍的可变长度特征处理，在实际工业界中其实是经常遇到的，而我们经常对其也只是简单的截断处理，很少有利用TensorFlow的相关特性去尽量保证特征信息的损失，因此这边文章也算是从某种程度上解决了上述问题。但是大家可以看到上面用到的方法都是在TensorFlow的contrib模块中，这意味着这个索引表方法可能还不是很成熟，并且我在实测中发现了一个问题，即使用了上述索引表后，在GPU训练环境下会报一些“segmentation fault”的错误，在CPU训练环境下是正常的。之前也跟TensorFlow社区的大佬交流过，应该是存在一些BUG在里面的，所以建议大家慎用哟~


#注：纯手工打造，实属不易，欢迎大家分享和转发~
#原创内容，转载需注明出处，否则视为侵权并将被追诉！

