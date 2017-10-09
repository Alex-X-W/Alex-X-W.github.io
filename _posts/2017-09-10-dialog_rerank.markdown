---
layout:     post
title:      "客服场景下由历史语料构建自动回复系统"
subtitle:   ""
date:       2017-10-09
author:     "Xuan"
header-img: "img/in-post/1/header.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - nlp
    - ml
    - bot
---


# 客服场景下由历史语料构建自动回复系统
*作者: 王玄*  


## 1. 引入
本文由作者在[助力来也](http://www.laiye.com)公司的工作总结而来. 着重讨论流程步骤, 不涉及具体代码本身.  
### 1.1 场景说明
客服-用户对话场景是一种很广泛常见的场景, 覆盖从售前到售后的全过程. 在传统上这一部分属于人力密集性, 需要投入对客服人员的前期培训以及运营精力. 自动回复系统的构建可以:  
* 减少客服工作压力提高效率, (在[助力来也](http://www.laiye.com)的实践中从1:100的客服-用户比提升到了1:400);  
* 标准化服务内容和质量. 企业可以积累以及构建知识库, 筛选出标准化的话术进行回复.  
* 构建"知识库-自动回复系统-人工标注"的良性闭环.  注意到自动回复系统到用户之间可以经过客服的缓冲, 这部分online labeling既保证了回复的质量, 又拥有了对数据的标注.  

### 1.2 自动回复系统总览
目前前沿的自动回复系统的实现是以"检索"与"重排"两个子系统级联的方式:  
![](/img/in-post/1/architecture.png)
图片来源: [AliMe Chat](http://www.aclweb.org/anthology/P17-2079)  

由检索(绿色部分)从知识库中第一级粗粒度筛选出一定数目的候选回复, 再交给重排(黄色部分)进行细粒度排序, 最后输出. 这样做的目的是为了在损失很少召回的前提下极大地提高了响应速度. 因为检索的速度很快, 而重排系统往往设计复杂且采用深层神经网络模型, 所以如果知识库里的全部回复都交由重排系统来排序, 响应速度会很慢.  

这里主要将重排系统.  

### 1.3 多轮重排系统
在实际的场景中, 经常有可能出现的情况是同一个目的的信息会通过好几句话才补充全. 类似的情况在taskbot中调用API时候需要用slot fitting策略来分轮次将API中的各个参数填补. 然而此时我们的场景是只拥有知识库(即历史上的问题和回答的日志), 并不能用slot fitting的策略来定. 所以如果我们要尽可能多获得上文的信息, 就需要不仅考虑最近的一句用户的utterance, 需要look further back, 也就是"多轮"的意义.  
我们的模型用tensorflow重写了[Sequential Matching Networks](http://www.aclweb.org/anthology/P17-1046).

### 1.4 名词/术语说明
名称 | 含义
--- | ---
*utterance* | 来自某一id的一句话. 我们的问题里一个*session*一般由一个用户和一个客服的多个来回的*utterance*组成
*session* | 一段完整会话  
*context* | 模型中考虑的所有上文utterances  
*response/answer* | 客服的utterance
  
  

## 2. 语料准备
### 2.1 清洗和整理
数据是historical dump data, 需要先进行清洗, 包括但不限于:  
* 去除 {非法字符, 时间戳/系统log/用户客服id等附加信息, ...}  
* 合并同一id连续发出的语句成一句话  
* 不同的session之间切分开(比如说用空行分割)

这里如果是数据量比较小的情况下, 在这一步可以做一些数据增强的工作, 比如粒度比较小(词/句)的一些shuffle. 

### 2.2 构造train/valid/test三个集合
#### 2.2.1. 根据模型考虑的上文utterance个数, 确定窗口大小, 滑动切分经过2.1步骤后的数据  
比如: 如果是考虑3轮会话, 则总共是3 * 2 = 6句utterance, 则窗口大小为6. 
#### 2.2.2. 混入负样本  
由于历史语料中我们假设所有的客服回复都是gold answer, 但是我们的系统是一个打分系统, 需要负样本才可以进行训练. 
对于每一个context, 随机从语聊中选择response与其进行拼接, 即构造出一个负样本. train set按正负1:1混合, valid和test按照1:9混合. train set按照1:1混合的目的是为了训练集的正负样本平衡. valid和test的混合方式根据业务实际来定, 比如说实际的模型会返回10个候选答案, 那么我们的召回就在10个里面, 所以选择1:9来进行混合. 当然知识库里面可能会有多个候选答案都可以作为答案, 这时的最后模型评价就需要人工的介入.  


## 3. 词表以及embeddings的准备
### 3.1 user-defined词表准备 
分词工具选用jieba, 在其上进行扩展.  
要务必注意注意: 
```
1)生成embeddings, 2)训练模型 以及 3)serving上线 时`jieba`的vocabulary务必保持一致.
```
具体地, 即在`import jieba`之后立即扩展同一份user-defined词表. 同时这份词表也可能也需要和上游的检索沟通, 确保分词结果的相容.  
user-defind词表里有领域词汇, 产品相关的词汇等.  

### 3.2 embeddings的准备
#### 3.2.1. 语料收集
收集领域相关文章, 并上我们的语料本身.
#### 3.2.2. 在训练好embeddings之后, 需要依次导出模型需要用的两个`_pickle`文件:  
名称 | 类型 | 内容 | 备注
--- | --- | --- | ---
`w2i` | python built-in `dict` | *word => idx* | 上线serving时仍然需要
`w2v` | 2-D `numpy array` | embeddings的具体值, 比如说以`np.float32`保存 | 上线serving时不需要

实际的`w2i`以及`w2v`比embeddings多两个向量, 分别对应`<PAD>`和`<UNK>`.  

## 4. 模型离线阶段
#### 离线阶段需要的基本代码
代码文件 | 说明 | 备注
--- | --- | ---
`train.py` | 从头开始的训练 | 单GPU
`continue_train.py` | 从checkpoint加载模型继续训练 | 单GPU
`export.py` | 从checkpoint中加载模型之后export | export之后要将生成的`saved_model.pb`和`variables/`置于一个名为version_number的文件夹下, 例如`1/`, 具体请见[TFserving Tutorial](https://www.tensorflow.org/serving/serving_basic)
  
上述的`train.py`和`continue_train.py`在考虑实际硬件资源是可以有多GPU的multi-towered版本, 加速训练.  

## 5. 模型上线阶段
这部分只需要熟悉下TFserving本身即可, server部分的代码可以套用tensorflow官方提供的ranking的serving部分, 只是client部分需要关注一下.  
仍然有一点需要重申:
```
1)生成embeddings, 2)训练模型 以及 3)serving上线 时`jieba`的vocabulary务必保持一致.
```

