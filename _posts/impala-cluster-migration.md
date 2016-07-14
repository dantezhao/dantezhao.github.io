---
author: zhao
title: 一次impala迁移记录
date: 2016-03-01 23:14:54
categories: data
tags: 
	- hdfs
	- impala
	- hadoop
description: 一次impala集群迁移，记录一下迁移时的思路以及遇到的坑。
---

## 前言

hadoop集群（应该说hadoop生态系统，现在spark、storm、kafka这些都包含在一起）迁移是很很多公司都很难避免的一个场景，个人理解，集群迁移主要分三方面：集群环境搭建、数据迁移、服务迁移。

新集群的搭建，不管是CDH或者是APACHE的版本，已经属于基本功范畴。

服务迁移又和具体的公司业务十分相关，这点应该是最麻烦的一点，需要和各个负责人确认。

数据迁移也是比较重要的一环，一般和数据比较相关的组件包括但不限于：hdfs、hbase、impala/hive、zookeeper、kafka。

下面将主要考虑impala的迁移，impala的迁移和hdfs最相关，因此两者在一起介绍。

## 迁移介绍

### 迁移场景

1.hadoop新老集群之间的迁移，由资源紧张或者配置较低的老集群向新集群迁移。

2.迁移需要数天或者数周才能完成。（需要测试新集群的稳定性，以及业务是否能正常运行）。

3.本文暂不考虑各种应用以及服务，只考虑hdfs和impala的数据迁移。

### 迁移方案

impala集群迁移主要有两部分内容：

1.hdfs中的数据。

2.impala中所有表和分区。

#### 全量迁移

在某一个时间点将现有的数据统一迁移一次，该阶段主要包括：

1.hdfs数据迁移。

2.impala数据库、数据表、分区的建立。

### 增量迁移

在进行全量迁移到新老集群完全迁移完成之前，定时进行增量迁移，可以考虑以一天为单位。

1.hdfs数据的增量迁移，相应目录的增量迁移。

2.impala分区增量添加。

## HDFS迁移

hive和impala大都会在hdfs上建外部表，因此需要先迁移hdfs相应数据。

hdfs的迁移主要使用distcp，写起来很简单啦，该命令在新老集群上都可以执行，具体的使用方式，看看官方文档就OK。

*注意*：

1.记得新建根目录。

2.在进行hdfs数据迁移的时候， 最好先进行时间的估算，根据集群的带宽情况估算一下迁移的总时间（大致的数量级）。然后尽量避免集群使用的高峰期来迁移。

## IMPALA迁移

### 准备工作

#### 外部表的数据文件

在hdfs迁移完成后相应的目录应该已经存在了，如果没有存在，执行impala建表语句时会报没有写权限的错误。

#### 老集群impala信息

1.所有的impala数据库：直接使用show databases。

2.所有的数据库建表语句（后两者我写了程序统一抓取）。

3.所有表的分区（在程序中拼接成相应的添加分区的语句）

附github地址：https://github.com/zhaodedong/CodeLibrary

程序里面的一些参数需要修改：impala地址，生成sql的路径。

## 补充

### 关键字字段

`show create table name`的时候，不会自动加那一个单引号。

因此在生成建表语句后，需要额外对一些关键字的字段处理，比如date，需要改为`'date'`

### 增量更新

增量更新的时候考虑新建impala分区

### 跳过sql错误

执行脚本的时候跳过sql错误，继续执行。

impala-shell -c -f jiaoben.sql

### 日志

建议执行的操作，将日志写入文件，方便查找哪些操作失败。

## 错误

### 权限错误

```
org.apache.hadoop.security.AccessControlException Permission denied:user=impala
```

impala建表的时候，这个错误很大程度上是在建表的时候location中的地址，在hdfs中不存在引起的。

建表的时候有些字段需要加``，比如date，需要改为`date`

