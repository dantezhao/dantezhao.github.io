---
layout: post
title:  "Scala使用SnakeYAML读取yaml文件"
categories: 编程语言
tags: scala yaml jvm
---

* content
{:toc}

## 前言

最近改进了一下程序，删除了大量的代码，改成可配置的情况，赶个时髦，配置的参数考虑使用yaml文件。

## scala读取yaml文件的内容

在网上找了一些内容，没有发现scala语言解析的yaml文件的包，那就只能用java的。

目前发现一个比较不错的包——SnakeYAML，遇到点坑，填了半个多小时，做个记录。




## SnakeYAML

java调用SnakeYAML的过程就不说，看文档比较easy。

scala调用SnakeYAML有一点小问题，对于初学者来说会有一个坑卡着。


错误1
```
class YamlTest {
}
object YamlTest extends App{
  val text2 = """
    name: you
    sex: male
              """
  val yaml = new Yaml
  val test = yaml.load(text2).asInstanceOf[Map]
  println(test.get("name"))
}
```

```
Error:(32, 44) type Map takes type parameters
  val test = yaml.load(text2).asInstanceOf[Map]
                                           ^
```

错误2

```
val test = yaml.load(text2).asInstanceOf[Map]
```

```
class MessageConfig {
  @BeanProperty var name: String = _
  @BeanProperty var sex: String = _
}

```

```
Exception in thread "main" java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to com.lagou.entity.MessageConfig
	at YamlTest$delayedInit$body.apply(YamlTest.scala:32)
	at scala.Function0$class.apply$mcV$sp(Function0.scala:40)
	at scala.runtime.AbstractFunction0.apply$mcV$sp(AbstractFunction0.scala:12)
	at scala.App$$anonfun$main$1.apply(App.scala:71)
	at scala.App$$anonfun$main$1.apply(App.scala:71)
	at scala.collection.immutable.List.foreach(List.scala:318)
	at scala.collection.generic.TraversableForwarder$class.foreach(TraversableForwarder.scala:32)
	at scala.App$class.main(App.scala:71)
	at YamlTest$.main(YamlTest.scala:15)
	at YamlTest.main(YamlTest.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
```

读yaml字符串

```
  val text2 = """
    name: you
    sex: male
              """
  val yaml = new Yaml
  val test = yaml.load(text2).asInstanceOf[java.util.Map[String, Any]]
  println(test.get("name"))

```

读文件

```
object YamlTest extends App{
  val file = "D:\\test.yml"
  val input = new FileInputStream(new File(file))
  val yaml = new Yaml
  val test = yaml.load(input).asInstanceOf[java.util.Map[String, Any]]
  println(test.get("name"))
}
```

## 注意

在解析的时候如果在yaml文件中存储了true、yes这些这两种关键字，解析的结果会把它变成B
oolean类型.......

不理解，即使设置了String，解析后还是Boolean。



******
2016-03-14 14:00:01 hzct
******
