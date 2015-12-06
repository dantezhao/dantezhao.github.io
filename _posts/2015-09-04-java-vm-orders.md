---
layout: post
author: zhao
title:  Java：虚拟机指令总结
date:   2015-09-04 20:14:54
categories: Java
---

* content
{:toc}


##前言

总结一些自己遇到过虚指令。

##规律

先说Java里面常见的数据类型：byte、short、int、long、float、double、char、reference。

每个数据类型抽取一个关键字母：b、s、i、l、f、d、c、a（这个比较特殊，不是r而是a）。

那么后面总结的指令就和这些关键字母相关了。比如说局部变量加载到栈的操作：iload、lload、fload、dload、aload，这些指令分别对应一种数据类型加载到栈中。

##指令

###加载指令

`(i,l,f,d,a)load_<n>`：将一个局部变量加载到操作栈。

`(i,l,f,d,a)store_<n>`：将栈顶的值存入局部变量表中。

`ldc #n` ：从常量池中取出内容推到栈顶。

`(i,l,f,d)const_<n>`：将一个常量值推送到栈顶，比如`iconst_1`是指将int型的数值1，推送到栈顶。

###运算指令

主要针对栈上的数值进行计算。

`(i,l,f,d)add`：对栈顶的两个元素进行相加，并将结果存入栈顶。

`iinc`：栈顶元素自增1。

###调用指令

`invokevirtual`：调用对象的实例方法，根据对象的实际类型进行分派(虚拟机分派)。

`invokeinterface`：调用接口方法，在运行时搜索一个实现这个接口方法的对象，找出合适的方法进行调用。

`invokespecial`：调用超类构造方法，包括实例初始化方法，私有方法和父类方法。

`invokestatic`：调用静态方法(static)。

`invokedynamic`：调用动态方法。



