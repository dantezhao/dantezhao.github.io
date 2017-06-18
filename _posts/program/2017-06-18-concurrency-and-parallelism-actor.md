---
layout: post
title:  "漫谈并发编程：Actor模型"
categories: 代码之熵
tags: Scala
---

* content
{:toc}

# 0x00 前言

一般来说有两种策略用来在并发线程中进行通信：共享数据和消息传递。熟悉c和java并发编程的都会比较熟悉共享数据的策略，比如java程序员就会常用到`java.util.concurrent`包中同步、锁相关的数据结构。

使用共享数据方式的并发编程面临的最大的一个问题就是数据条件竞争（data race）。处理各种锁的问题是让人十分头痛的一件事。

> 和共享数据方式相比，消息传递机制最大的优点就是不会产生数据竞争状态（data race）。实现消息传递有两种常见的类型：**基于channel的消息传递和基于Actor的消息传递**。本文主要是来分享Scala的Actor模型。




### 文章结构

本篇博客尝试讲解Actor模型。由于目前Scala的使用频率较高，因此主要语言为Scala

主要分为下面几个部分：

1. Actor模型的基本概念使用
2. 讲一下akka框架中scala的基本使用，主要提几个重要的api
3. 写几个例子帮助理解
    1）. HelloWorld 简单版：通过这个例子来简单看一下Akka中Actor的使用
    2）. HelloWordl 进阶版：稍微进化了一点点，多了preStart和PoisonPill的使用。
    3）. WordCount 伪分布式：一个单机版的wordcount，一个map，多个reduce。后续再补充完全分布式的程序。


# 0x01 基本概念

Actor是计算机科学领域中的一个并行计算模型，它把actors当做通用的并行计算原语：一个actor对接收到的消息做出响应，进行本地决策，可以创建更多的actor，或者发送更多的消息；同时准备接收下一条消息。

在Actor理论中，一切都被认为是actor，这和面向对象语言里一切都被看成对象很类似。但包括面向对象语言在内的软件通常是顺序执行的，而Actor模型本质上则是并发的。

## 什么是Actor模型

> Actor的概念来自于Erlang，在AKKA中，可以认为一个Actor就是一个容器，用以存储状态、行为、Mailbox以及子Actor与Supervisor策略。Actor之间并不直接通信，而是通过Mail来互通有无。

每个Actor都有一个(恰好一个)Mailbox。Mailbox相当于是一个小型的队列，一旦Sender发送消息，就是将该消息入队到Mailbox中。入队的顺序按照消息发送的时间顺序。Mailbox有多种实现，默认为FIFO。但也可以根据优先级考虑出队顺序，实现算法则不相同。

## 消息和信箱

异步地发送消息是用actor模型编程的重要特性之一。消息并不是直接发送到一个actor，而是发送到一个信箱（mailbox）。如下图。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/actor_1.png?x-oss-process=style/blog_dantezhao)

*这样的设计解耦了actor之间的关系——actor都以自己的步调运行，且发送消息时不会被阻塞。虽然所有actor可以同时运行，但它们都按照信箱接收消息的顺序来依次处理消息，且仅在当前消息处理完成后才会处理下一个消息，因此我们只需要关心发送消息时的并发问题即可。*


# 0x02 Akka中的Actor

我们会用到Akka框架提供的Actor，因此在这里先大致介绍一下Akka中的Actor使用方式。

## Actor System

Actor System是进入AKKA世界中的一个入口，也可以看做是Actor的系统工厂或管理者，掌控者Actor的生命周期，包括创建、停止Actor，当然也可以关闭整个ActorSystem。

比如我们后面会展示出来的代码：

```scala
object BetterHelloWorld extends App{
  val system = ActorSystem("HelloActors")
  system.actorOf(Props[BetterMaster], "master")
}
```

## Actor的层级

Actor的整个体系就像是一家层级森严的企业组织，层次越高，管理权限与职责就更大。在AKKA中，parent actor就是child actor的supervisior，这意味着parent actor能够掌控child actor的整个生命周期。而这种分级的模式也能够更好地支持系统的容错。

如果要创建child actor，就不再调用ActorSystem的actorOf()方法。

如下`BetterMaster`就是一个parent actor，而BetterTalker就是一个child actor，完整代码请看后面的例子。

parent actor：

```scala
class BetterMaster extends Actor {
  val talker = context.actorOf(Props[BetterTalker], "talker")
  override def preStart { ... }
  def receive = { ... }
}
```

child actor：

```scala
class BetterTalker extends Actor {
  def receive() = {
    ...
  }
}
```

*补充：* AKKA提供的方法是在Parent Actor内部，通过调用ActorContext的actorOf()方法来创建它自身的child actor。

## Actor的生命周期

AKKA为Actor生命周期的每个阶段都提供了一个钩子（hook）方法，我们可以通过观察自定义Actor需要重写的方法来理解Actor的生命周期。

Actor被定义为trait，其中一个典型的方法对是preStart()与postStop()，顾名思义，两个方法分别在启动和停止时被调用。 这两个方法可以看我们在后文中的例子。

Akka官方文档提供了说明Actor生命周期的图片，如下所示：

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/actor_2.png)


官网的图不好看，盗一张感觉不错的。大致能明白就好。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/actor_3.png)

除了正常的start和stop操作，我们也会在例子中用到preStart()、postStop()、PoisonPill等操作。

# 0x03 例子

下面会放三个例子，分别有不同的知识点。

1. HelloWorld 简单版：通过这个例子来简单看一下Akka中Actor的使用
2. HelloWordl 进阶版：稍微进化了一点点，多了preStart和PoisonPill的使用。
3. WordCount 伪分布式：一个单机版的wordcount，一个map，多个reduce。后续再补充完全分布式的程序。


## 1. HelloWorld 简单版

简单的Hello，代码可以看注释。

```scala
package com.dantezhao.helloworld.simple

import akka.actor._
/**
  * Created by Dante on 2017/6/17.
  */
object SimpleHelloWorld extends App{

  val system = ActorSystem("HelloActors")
  val talker = system.actorOf(Props[SimpleTalker], "talker")
  //发送三条消息
  talker ! SimpleGreet("Dante")
  talker ! SimplePraise("Winston")
  talker ! SimpleCelebrate("clare", 18)
}

//这里用三个case class来声明三种消息类型
// case class有一个好处就是可以用在case语句中
case class SimpleGreet(name: String)
case class SimplePraise(name: String)
case class SimpleCelebrate(name: String, age: Int)


//这是我们第一个actor
//它会接收三种消息并打印相应的输出
class SimpleTalker extends Actor {
  def receive = {
    case SimpleGreet(name) => println(s"Hello $name")
    case SimplePraise(name) => println(s"$name, you're amazing")
    case SimpleCelebrate(name, age) => println(s"Here's to another $age years, $name")
  }
}
```

运行看一下结果：

```
Hello Dante
Winston, you're amazing
Here's to another 18 years, clare
```

## 2. HelloWorld 升级版

在此版本加上了控制Actor结束的PoisonPill。

可以看到当接收到PoisonPill，Actor将不再接收数据。

```scala
package com.dantezhao.helloworld.better

import akka.actor.{Actor, ActorSystem, PoisonPill, Props, Terminated}

/**
  * Created by Dante on 2017/6/18.
  */
object BetterHelloWorld extends App{
  val system = ActorSystem("HelloActors")
  system.actorOf(Props[BetterMaster], "master")
}

//这里用三个case class来声明三种消息类型
// case class有一个好处就是可以用在case语句中
case class BetterGreet(name: String)
case class BetterPraise(name: String)
case class BetterCelebrate(name: String, age: Int)

class BetterTalker extends Actor {

  def receive() = {
    case BetterGreet(name) => println(s"Hello $name")
    case BetterPraise(name) => println(s"$name, you're amazing")
    case BetterCelebrate(name, age) => println(s"Here's to another $age years, $name")
  }
}

class BetterMaster extends Actor {

  val talker = context.actorOf(Props[BetterTalker], "talker")

  override def preStart {
    context.watch(talker)

    talker ! BetterGreet("Dante")
    talker ! BetterPraise("Winston")
    talker ! BetterCelebrate("Clare", 16)
    //发送一个毒丸，告诉actor已经结束了。因此后面发送的消息将不会被传递
    talker ! PoisonPill
    talker ! BetterGreet("Dante")
  }

  def receive = {
    case Terminated(`talker`) => context.system.terminate()
  }
}

```

运行结果。

```
Hello Dante
Winston, you're amazing
Here's to another 16 years, Clare
[INFO] [06/18/2017 11:32:12.309] [HelloActors-akka.actor.default-dispatcher-3] [akka://HelloActors/user/master/talker] Message [com.dantezhao.helloworld.better.BetterGreet] from Actor[akka://HelloActors/user/master#-1817351260] to Actor[akka://HelloActors/user/master/talker#-385284367] was not delivered. [1] dead letters encountered. This logging can be turned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.

```

## 3. 伪分布式 WordCount

写一个模拟WordCount的程序，工作流程如下：

1. MasterActor负责管理其余的Actor，并发送数据给MapActor。这里的发送数据暂时随意发几条。
2. MapActor接收到一行行的数据后，将数据处理成`(word:1)`的形式，并发送到所有的ReduceActor中。
3. ReduceActor接收到数据后，将数据处理成`（word:count_num）`的形式，发送给AggregateActor。
4. 最后，AggregateActor汇集ReduceActor的数据，并打印出top10。

如图，是Actor的结构。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/actor_4.png?x-oss-process=style/blog_dantezhao)

这里不再贴完整的代码，只放出来几个关键的代码段，其余的请参考github。

MapActor的处理逻辑：

```scala
 // 把一句话切割，返回（word:1）
  def splitLine(line: String): MapData = {
    var dataList = new ArrayBuffer[Word]
    var parser: StringTokenizer = new StringTokenizer(line)
    while (parser.hasMoreTokens()) {
      var word: String = parser.nextToken().toLowerCase()
      dataList.append(new Word(word, 1))
    }
    return new MapData(dataList)
  }
```

ReduceActor的处理逻辑：

```scala
def reduce(dataList: ArrayBuffer[Word]): ReduceData = {

    val reducedMap = new HashMap[String, Integer]
    for (wc: Word <- dataList) {
      val word: String = wc.word
      if (reducedMap.contains(word)) {
        reducedMap.put(word, reducedMap(word)+1 )
      } else {
        reducedMap.put(word, 1)
      }
    }
    return new ReduceData(reducedMap)
  }
```

AggregateActor的处理逻辑：

```scala
def aggregateInMemoryReduce(reducedList: HashMap[String, Integer]) {
    var count: Integer = 0
    for (key <- reducedList.keySet) {
      if (wordCounts.contains(key)) {
        count = reducedList(key) + wordCounts(key)
        wordCounts.put(key, count)
      }
      else {
        wordCounts.put(key, reducedList(key))
      }
    }
  }
```

最后的求top10的逻辑：

```scala
override def postStop() {
    wordCounts.toList.sortBy(_._2).takeRight(10).map(println)
  }
```

运行看一下结果：

```
(best,1)
(brown,1)
(family,5)
(belong,5)
(same,5)
(to,6)
(and,6)
(fox,6)
(dog,8)
(the,8)
```

# 0x04 总结

本来是想搞Go的并发编程的，但是CSP和Actor模型有很多接近的地方，想搞CSP模型，总是要看Actor的，因此就先花了点时间看一下Actor模型。

目前来讲，也是学着写着，比较遗憾的是没有实际的项目支撑，理解程度还是不深。 代码能正常运行，但是不是很理想，最后没搞出来分布式的程序。 只能留待以后有兴趣了再搞了。

完整代码地址：https://github.com/dantezhao/concurrency_and_parallelism

# 参考

- http://dyingbleed.com/akka-1/
- https://github.com/liubin/programming-scala/blob/master/content/15_actors/00_preface.md
- https://zhangyi.gitbooks.io/akka-in-action/content/actor.html
- https://www.infoq.com/news/2014/10/intro-actor-model
- https://media.pragprog.com/titles/pb7con/Bonus_Chapter.pdf

***
2017-06-18 21:47:00 wxxy
