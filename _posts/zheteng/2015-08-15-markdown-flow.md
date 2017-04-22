---
layout: post
title:  "Markdown画流程图"
categories: 不折腾被变羊
tags: Markdown
---

* content
{:toc}

## 如何使用markdown画流程图

话说网上关于使用markdown画流程图的相关的教程真是一堆堆的坑，我丝毫不怀疑这些博文作者确实是知道如何使用markdown画流程图了，但是写出来的文章却丝毫没有明白具体的使用方法，于是有了这篇学习笔记。




## 如何画流程图

### 流程图画在哪了

先说一下网上我搜的那些博客都没有说清楚的一点，流程图在markdown里面的体现形式。

在markdown语法中，流程图的画法和代码段类似，也就是说，流程图是写在两个` ``` ` 之间的。

比如说Java代码，会是这样一种格式：` ```java `  代码段 ` ``` `

那么流程图就是这样的：` ```flow `  代码段 ` ``` `

此处不再截图，自从开始用markdown写博客，能不使用图片，基本上就不再传图了。

### 画流程图的步骤

流程图的语法大体分为两段

1. 第一段用来**定义元素**

2. 第二段用来**连接元素 **

定义元素阶段的语法是

tag=>type: content:>url

tag就是一个标签，在第二段连接元素时用 type是这个标签的类型

### 流程图的基本类型

目前在画流程图的时候我知道大概有这六种基本类型。

```
start
end
operation
subroutine
condition
inputoutput
```

**开始**

st=>start: 开始

**操作流程**

st->op->cond

**条件**

cond=>condition: 确认？

**结束**

e=>end: 结束


## 示例

### 代码


```flow
st=>start: Start
e=>end
op=>operation: My Operation
op1=>operation: My Operation1
cond=>condition: Yes or No?

st->op->cond
st->op1
cond(yes)->e
cond(no)->op
```

### 结果

```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```


***
2015-08-15 23:14:54 于 bh xzl
