---
layout: post
author: zhao
title: The Data Revolution Speaker（Hadoop之父Doug Cutting在清华的讲座）
modified: 2015-08-17
tags: [Meeting]
image:
  feature: pic-1.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

##The Data Revolution Speaker 讲座记录

>2014-12-12 14:30     清华大学FIT楼二路多功能厅

整个讲座约一个小时，两点半左右开始，前半个小时左右Doug Cutting 总共大概7张PPT，后半个小时互动。

##**演讲内容**

Doug Cutting总共讲了大概7张PPT，PPT里面没什么内容，每张PPT只有一个标题，正文是一张图片，内容主要讲的是自己的开源事业、lucene、hadoop等。

**PPT One**：Means For Change ： Hardware

提了moore定律，讲了处理器、存储这些硬件更新的速度很快。这是一个硬件基础。

**PPT Two**：Fuel For Change : Data

这里讲了一个逻辑，引出来了Open Source的重要性。

首先提出来Software is eating the industry，软件飞速发展；由此会产生各种各样的数据，而且数据量非常大，价值非常高；因此需要有Tools来处理这些数据，继而引出了下一张PPT：OpenSource。

**PPT Three**：Seeds For Change ：Open Source

关于开源软件的好处大概讲了一下，没有讲特别多，大致上也是方便开放，有用故而用之。

其中提到他自己开始开源事业的一个想法，就是在做lucene的时候，发现自己不适合搞Business，所以give it away~~

这张ppt还提到三个重要的component，没有听清是什么的三个组成部分，大概是整个计算机行业的？

三个分别是：Hardware、Data、Software

**PPT Four**：New DataStyle：Hadoop

这张PPT引出来了Hadoop，Hadoop大概介绍了一下。提到了GFS，hadoop的很多思想都是参考了gfs的。Google发表了论文，提出了它的这种理论，大家都很感兴趣，但是不是Google的原因，因此没法非常方便用。这时候Hadoop就出来了，OpenSource方便，易得。有其天然的亲民优势。

Doug Cutting提到自己去了Yahoo，因为Yahoo需要处理大量的数据，还有大量的硬件可以用，和自己很契合。

**PPT Five**：Style Catches on：Ecosystem

介绍了Hive、pig、spark等，没过多的讲。

**PPT Six**：Victor Emerges：Enterprise Data Hub

大致讲了自己在Cloudera工作，介绍了Enterprise Data Hub的重要。记得说了一句话： I am lucky in the right place in the right time.（语法感觉有点别扭）提到了这是future tool。

**PPT Seven**：The Data Multi-Tool

快结束了，说到了hadoop的一些存在意义，举了一个例子，这个例子正是PPT的图片，是个手机。大致意思是：手机可以干很多事，比如照相，但是照相的功能不如一些专业的相机。但是有一点可以确定，大家用手机照相的时间比相机多，为什么呢，因为手机一直在你身边，你什么时候都可以用，而且除了照相，我还可以把照片分享，总的来说，就是已经存在，而且方便。

Hadoop也类似，现在有很多的计算框架，Spark、Storm这类的。这种情况不必否认其他的存在，hadoop大家会比较熟悉，而且应用很广泛，在你需要的时候，可能你就有一个hadoop的集群环境，有些计算可能Spark性能更好，但是hadoop也可以做，方便使用。

>这让我想到了操作系统，未必是windows最好，但是大家都习惯了，也就是够用了，再出现一个新的操作系统，除非你让我感觉有了你我就不想用windows了，windows已经够用了，不必非要把它换掉，类似道理。

##**问答内容**

最后是提问时间，大该记录了几个问题：

**1.安全问题。**

Doug Cutting回答的大概意思是：技术解决+Social Solution。

**2.relational database和 nosql**

这个其实不是新问题了，Doug Cutting说的一句重点：**each has its uses**

**3.spark，storm的存在**

比如spark是用memory的，hadoop现在是hdfs，是否要向spark学习一下呢

Doug Cutting的大概回答是，这是ecosystem，每个component都有其作用，各善其职即可，**I am happy to see spark**。还有就是，这是开源软件，并不是一个公司控制了hadoop另一个控制spark，两个公司在竞争。因为是开源，最终的目的都是为大家所用。

**4.什么是bigdata**

Doug Cutting回答了很长一串，最后听出来重点是：**Not the size，it's the style。**

>喏，bigdata是一种思想，一种处理方式上的体现。我是否可以理解为数据多少不重要，重要的是处理的方法？

**5.Cloudera和Hortonworks**

 Doug Cutting也回答了一些客套的话，然后说的是：**Happy competition。**


另外：提问送书。走的晚一点，可以找Doug Cutting本人签字和合影。

Doug Cutting人很好，非常和气，另外特别高，一米八左右感觉到他下巴左右，压力太大，他在签字的时候是屈膝跪坐在地上的，看的我很感动。

书上写了“Enjoy hadoop和自己签名。”

