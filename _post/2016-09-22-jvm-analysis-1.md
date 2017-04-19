---
layout: post
title:  "一次jVM性能调优记录"
categories: 代码之熵
tags: jvm
---

* content
{:toc}

## 前言

填别人留下来的坑其实挺无奈的，会被搞的特别烦，特别是我这种要填三四个人留下来的坑的时候，满满的都是无奈。

幸好的是填坑也可以选择一种更能提升自己的方式来填。

这次遇到的一个程序，是一个从kafka消费并且插入mysql的程序，该程序历经三人之手，频频出问题，一直没有被解决。传到现在，症状是这样的：该程序跑个两三天后会莫名其妙的停止消费，不再插入数据了，据说也不报错，进程还在，反正就是不干活了。




## 分析过程

等了一天半，这个程序不出意料的出问题了，状况依然是停止消费，不再插入数据。我不太清楚之前的人是怎么排查的，暂时按照我自己的方式来摸索。


首先看一下jvm内存的分布情况，一下子就看到点子上了。old区和perm区基本上都满了，然后可以观察到每隔几秒钟就会进行一次full gc。这样的话，整个程序不是一直都在full gc了，哪还有心情干别的活？

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-1.png)

抓紧时间看一下程序的报错，非常明显的OOM问题，不晓得之前的童鞋找到这没。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-2.png)



按理说看到程序出错了就该赶快补数据了，但是为了抓住错误的本质，我只能顶住一些压力，缓半个小时再说，抓紧时间看一下各个指标。

首先是top一下看看cpu和内存，pid是25729那只，貌似压力很大的样子。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-3.png)

然后是jmap看一下各个区的情况。仔细看一下就会发现，在Old Generation区已经占用了2.6G的大小了，这个应该是十分不正常的。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-4.png)

这时候第一反应就是看一下垃圾回收用的什么，嗯，ParallelGc，按说也没啥问题啊，我可能也就是用这个了。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-5.png)

这时候无意间发现了jstat的一个参数`gccause`，功能是查看最近一次GC统计和原因。发现是Allocation Failure，和猜测的差不多。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-6.png)

然后我就想导出堆栈的信息，然后分析一下，但是太大了，有4G的大小，我的笔记本根本打不开。所以只能先用jcmd看一下，就发现了了一些比较好玩的东西。排在前面的这俩JDBC是啥么个意思。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-7.png)

其实追到这里基本已经可以定位到程序里面的问题了，和关闭statment有关。

但是我们不能到此就结束，为了证明我是爱学习的，我用eclipse的的可视化分析工具，打开了jmap导出的文件，具体怎么导出的可以参考这篇：http://zhaodedong.com/2015/09/01/jvm-tools/。

有个工具就是方便很多，首先就可以定位到到底是哪里占用了这么的内存。然后就追呀追，果真还是学到了点东西。


![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-8.png)

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-9.png)

看下面这个图，首先程序里面是用的inset语句，但是会有很多的ResultSet对象，然后我就挨个进去看了ResultSet里面有什么东西，发现都是要插入的数据，我一直以为JDBC只有在获取结果的时候才会用ResultSet，没想到插入数据同样用的这个。

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/jvm-analysis-10.png)


## 总结

jdk的各种工具我一直都有笔记做过记录，但是没有做过比较多的分析，趁此机会就详细的挨个用了一遍。

个人感觉，其实在分析问题的时候还是需要一定的敏感度的，光知道工具是没有用的，必须还要有对问题的敏感度，能够明白每个工具获得的结果有什么用，哪些结果数据是不正常的。这点其实十分重要，至少我在分析之初是很懵的，看着一堆数据完全不知道怎么下手，后来看了不少的文章也问了一些同事之后才把这些搞明白。




## 补充

本着尝试的原则，我也尝试jvm参数调优实践。发现还是有效果的，至少多坚持的更久的时间程序都没有出问题。但是由于没有抓住问题的核心错误，依旧不是解决问题的方法。

```
nohup java -classpath $CLASS_PATH com.***.Consumer -Xms1024M -Xmx6000M -XX:PermSize=128M -XX:MaxPermSize=256M -XX:+UseParallelGC XX:+UseParallelOldGC &
```

***
2016-09-22 00:14:00 rljp


***

转载请注明： 转载自赵德栋的 [**个人主页**](http://zhaodedong.com) [**CSDN博客**](http://zhaodedong.com)

作者：赵德栋，[作者介绍](http://zhaodedong.com/about/)

本博客的文章集合：http://zhaodedong.com/category/
