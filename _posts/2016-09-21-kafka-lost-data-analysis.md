---
layout: post
title:  "kafka丢数据、重复数据分析"
categories: 数据平台
tags: kafka
---

* content
{:toc}

## 前言

填别人的坑again。数据不正常，追到kafka这里了，分析了很久的程序，做一个总结，关于丢数据和重复数据。


## 丢数据

先说丢数据。目前遇到一种丢数据的情况。

如果`auto.commit.enable=true`，**当consumer fetch了一些数据但还没有完全处理掉的时候，刚好到commit interval出发了提交offset操作，接着consumer crash掉了。这时已经fetch的数据还没有处理完成但已经被commit掉，因此没有机会再次被处理，数据丢失。**

那么换一种方法，我们手动提交offset，这时会出现另外一种重复数据的情况，后面会提到。

## 重复数据

假设一个场景，我们使用High Level Consumer API来消费kafka中的数据，每当消费N条数据并成功插入Mysql后，我们手动执行一下`consumer.commitSync()`操作，提交一次offset。这种情况是先消费成功后再提交，因此不再丢数据了，但是会出现重复数据的情况。

但是！**如果当程序消费了一定数量的数据之后，还没来得及提交offset，程序crash了，那么下次再启动程序之后，就会重复消费这部分数据。**如果程序不够稳健，是可以出现十分多的重复数据的。

## 总结

以上自己的分析都是使用的high level api，因为简单嘛，因此对于offset的控制相对来说比较粗粒度。据说low level的控制力度比较细，可以比较好地控制offset。但是使用比较复杂，暂时还没有打算用。

另外，spark对于kafka的调用也有比较好的实现，据说是能比较好的控制数据丢失的问题。目前还没有研究完，后面继续。

<font color=#0099ff face="黑体">color=#0099ff size=72 face="黑体"</font>

## 参考

- http://www.zhangrenhua.com/2016/08/02/hadoop-spark-streaming%E6%95%B0%E6%8D%AE%E6%97%A0%E4%B8%A2%E5%A4%B1%E8%AF%BB%E5%8F%96kafka%EF%BC%8C%E4%BB%A5%E5%8F%8A%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/
- http://www.fwqtg.net/kafka-consumer%E9%98%B2%E6%AD%A2%E6%95%B0%E6%8D%AE%E4%B8%A2%E5%A4%B1-%E5%8D%9A%E5%AE%A2%E5%88%86%E7%B1%BB%EF%BC%9A-kafka-kafkaoffset-commit.html
- http://www.cnblogs.com/fxjwind/p/3810740.html

***
2016-09-21 20:39:00 hzct

***

转载请注明： 转载自赵德栋的 [**个人主页**](http://zhaodedong.com) [**CSDN博客**](http://zhaodedong.com)

作者：赵德栋，[作者介绍](http://zhaodedong.com/about/)

本博客的文章集合：http://zhaodedong.com/category/
