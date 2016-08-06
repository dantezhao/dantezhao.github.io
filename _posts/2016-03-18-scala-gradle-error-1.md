---
layout: post
title:  "scala,gradle报错：'jvm-1.8' is not a valid choice for '-target'"
categories: 编程语言
tags: jvm scala gradle
---

* content
{:toc}



## 开发环境：

- java：1.8
- scala：2.10.6
- gradle：2.11
- idea：15

## 使用gradle构建scala的项目报错

```
Error:scalac: 'jvm-1.8' is not a valid choice for '-target'
Error:scalac: bad option: '-target:jvm-1.8'
```




## 错误原因

在stackoverflow中的回答是：

>Gradle by default use the ant task to build Scala code, and https://issues.scala-lang.org/browse/SI-8966 shows that jvm 1.8 was not added as a supported target until Scala 2.11.5.

>You can try using the Zinc based compiler with by adding the following the gradle build file.
```
    tasks.withType(ScalaCompile) {
      scalaCompileOptions.useAnt = false}
```
>You may also need to add the zinc compiler to your dependency list.

## 解决方法

解决方法有三个：

1.升级scala版本，把scala升级到2.11就OK了
2.降低java版本，1.7不会出错
3.老外的方式，没做尝试

## 补充
修改gradle的版本没有效果，其它的构建工具不会出现这个错误


***
2016-03-18 19:08:00 hzct
