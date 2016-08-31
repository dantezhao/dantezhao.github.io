---
layout: post
title:  "Impala实践之十五：Impala使用文档"
categories: 数据平台
tags: impala shell
---

* content
{:toc}
## 前言

由于前期大家使用Impala的时候都比较随意，再加上对Impala的原理不清楚，因此在使用的过程中对Impala带来了很大的压力。

经过前段时间的研究和实验。我整理了一份Impala使用文档，供组内小伙伴使用。




## 概述

针对大数据集群Impala组件的使用说明。包括使用原则、建议和规范。以下所有建议均建立在日常使用过程中总结的经验和实际测试结果之上。若有问题，请联系文档发布者。

### 读者对象

- 平台数据开发人员
- 平台数据分析人员
- 数据挖掘研发人员

## 元数据操作规范

### 总体说明：

1. 只有通过hdfs增加或删除分区中文件后，才需要人为更新元数据，其余情况依赖impala自带更新机制即可。

2. 通过hdfs增加或删除分区中文件后一律使用refresh tablename操作，性能损耗最低。

3. 日常查询操作一律不加-r参数。如果出现提示元数据过期（该提示为目前版本bug，不必理会），可断开重连或者使用refresh操作。

**注意：** 如果在同一个shell脚本中，先执行了ddl操作，然后又对相应的库执行查询，会出现元数据同步延迟导致无法读取信息的操作。


###	refresh [tablename]（部分操作使用）

使用场景：

1. 通过HDFS添加或删除分区下文件

使用规范：

1. 通过hdfs在分区中添加和删除文件后，使用refresh tablename操作

2. 其余任何操作不需要使用。

使用建议：

1. 如果提示某张表元数据可能过期时，可以断开impala-shell后重连或者使用该语句。

2. 日常impala任务，可以取消脚本中的-r参数，使用refresh tablename代替。

3. 在查询端，不需要执行任何元数据相关的操作。

### -r参数（不使用）

该操作等同于`invalidate metadata`。

使用场景：

1. 通过Hive修改元数据

2. 通过HDFS添加或删除分区下文件

使用规范：

**个人任务脚本中不使用**

**查询任务时不使用**

使用建议：

除非在短时间内通过hdfs或者hive修改了大量的元数据，否则不要使用。


### `invalidate metadata`（不使用）

使用场景：

1. 使用场景同-r参数

使用规范：

个人任务脚本中不使用

查询任务时不使用

使用建议：

1. 使用hive进行表ddl操作后需要执行invalidate metadata tablename


## `invalidate metadata [tablename]`（少使用）

使用场景：

1. 同`refresh tablename`

使用规范：

1. 一律使用refresh操作代替该操作。

2. 在shell脚本（使用impala）中如果先执行ddl操作，而且需要立即执行查询或者插入操作，需要在查询或者插入操作执行`invalidate metadata tablename`操作。

## 负载均衡

总体说明：

1．大批量的任务以后通过负载均衡
2．Jdbc和impala-shell分别使用相应的连接端口

### Jdbc任务

使用举例：

`jdbc:hive2://host:25003/;auth=noSasl`

###　Impala-shell任务

使用举例：

通过shell执行的脚本，统一使用impala统一提交脚本

## Impala统一提交脚本使用说明

为便于impala提交任务管理，以及集群节点故障后保证impala提交任务能正常运行，以后Impala的shell任务统一使用平台提供的脚本。

### 脚本功能描述

统一分配impala连接信息。

统一分配mysql连接信息。

### 脚本位置：

调度系统：`/path/get_ds_conf.sh`

提交机：`/path/get_ds_conf.sh`

### 使用说明

**Impala使用举例**

```
#!/bin/bash 
source /path/get_ds_conf.sh
#获得配置数据
IMPALA_COMMOND=`getImpalaCommand`
${IMPALA_ COMMOND }  "select * from table limit 100"
```

**Mysql使用举例**

```
#!/bin/bash 
source /path/get_ds_conf.sh
MYSQL_COMMOND=`getMysqlCommand`
${MYSQL_ COMMOND }  "select * from table limit 100"
```

## 总结

文档在四五月份的时候就整理了，今天突然想起来这茬了，就整理处理，从word里面摘出来一部分内容。


***
2016-08-31 15:40:00 hzct
