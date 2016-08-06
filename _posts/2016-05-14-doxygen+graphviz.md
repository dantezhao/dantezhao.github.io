---
layout: post
title:  "如何使用doxygen+graphviz阅读源码"
categories: Technique
tags: 源码 doxygen
---

* content
{:toc}

## 0x00 前言

>工欲善其事必先利其器。

**问：** 如何快速地把握一个开源项目的脉络？

**答：** 除了其它阅读源码的一些技巧外，最好要经常看类调用关系图。

**问：** 怎样看一个项目的类调用关系图？

**答：** 两种常用方式：1. 常用IDE都带有查看uml图的功能，比如idea和myeclipse。2. 使用其它专门用来看代码的工具，比如这次我要推荐的doxygen

**问：** doxygen有什么用？

**答：** doxygen主要有下面两个功能：1. 看文档，它类似与javadoc，但是功能更全一些，而且不限于java一种语言2. 看类的各种关系图，非常细致，非常实用，用了就知道。

**问：** doxygen怎么用？

**答：** 别急，下面就将专门介绍。

## 0x01 安装doxygen+graphviz

### 环境

- 操作系统：ubuntu 16.04
- 阅读源代码：junit4.12
- 浏览器：chrome

### 安装

直接用apt安装就好，挺省事的。喜欢折腾就去源码安装吧。

```
sudo apt install doxygen //主程序
sudo apt install doxygen-gui //doxygen的gui版
sudo apt install graphviz //生成类图用
```

## 0x02 使用doxygen生成文档

因为要阅读junit的代码，这次就以junit4.12的代码为例，详细说一下如何来配置doxygen。

### 1.启动doxygen

咱们就用图形化界面的方式来搞，如何？

```
doxywizard
```

### 2.配置doxygen

第一步：基本设置。（这个我就不详细说了，源码位置以及生成目录的位置，能看懂英文应该就能选对。）


![](http://obg1rl2km.bkt.clouddn.com/doxygen-basic-config.png)


第二步：选择生成调用关系图

![](http://obg1rl2km.bkt.clouddn.com/doxygen-generate-graph.png)

第三步：高级设置

在Expert->build下设置：
![](http://obg1rl2km.bkt.clouddn.com/doxygen-expert-config.png)

在Expert->dot下设置：
![](http://obg1rl2km.bkt.clouddn.com/doxygen-export.png)

第四步：生成

最后点击“run doxygen”就可以生成了。
![](http://obg1rl2km.bkt.clouddn.com/doxygen-run.png)

## 0x03 阅读文档

阅读文档就不细说了，大家一看就明白了，在这贴几个图看看效果。

![](http://obg1rl2km.bkt.clouddn.com/doxygen-doc.png)

![](http://obg1rl2km.bkt.clouddn.com/doxygen-more-graph-1.png)

![](http://obg1rl2km.bkt.clouddn.com/doxygen-more-graph-2.png)

很爽吧，看源码看蒙的时候，就来参照一下这些图，就更轻松了。

## 0xFF 总结

doxygen只是一个工具，其实它对于看源码只是起到了一个帮助作用。能帮你更优雅地理解整个项目的结构。至于能不能看懂源码，就是自己的事了。

愿君共勉！！！

******
2016-05-14 11:34:00 hnds
