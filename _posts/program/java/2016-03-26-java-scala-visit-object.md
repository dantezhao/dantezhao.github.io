---
layout: post
title:  "Java和scala互操作之不能读取嵌套object"
categories: 代码之熵
tags: Java Scala
---

* content
{:toc}

## 前言

由于一些历史原因，一部分java代码还没有完全迁移至scala。因此存在了不少java和scala互操作的代码。这次又碰到一个小的问题。




## 举个栗子

### scala代码

```
/**
  * Created by Dante on 2016/3/26.
  */
object Property {
  val scalaConfig1 = "hello this is out"

  object ScalaConfig {
    val scalaConfig2 = "hello this is in"
  }
}
```

### java代码

```
/**
 * Created by Dante on 2016/3/26.
 */
public class JavaScalaTest {

    public static void main(String[] args) {
        String s = Property.ScalaConfig.scalaConfig2();
        System.out.println(s);
    }
}
```

### 错误

当java读取scala嵌套在object中的object的val时，会报错，找不到符号。

```
Error:(11, 28) java: 找不到符号
  符号:   变量 ScalaConfig
  位置: 类 com.lagou.scala.util.Property

```
修改一下，读取object外面的val才可以。

```
public class JavaScalaTest {

    public static void main(String[] args) {
        String s = Property.scalaConfig1();
        System.out.println(s);
    }
}
输出：
hello this is out
```

***
2016-03-26 12:00:00 hzct
