---
layout: post
title:  "Impala实践之二：Hive元数据"
categories: 数据平台
tags: impala hive 元数据
---

* content
{:toc}

## 0x00 前言

深入学习Impala的最主要一个原因就是目前在使用Impala的时候遇到了各种了性能问题，之前定位过一次问题，猜测其性能损耗的一个主要原因在`INVALIDATE METADATA`和`-r`参数上，但是对此并不是十分理解，因此需要深入一点底理解这些概念，方面更准确地定位问题。

下面将从三个角度来分析Impala元数据：Hive元数据库、`INVALIDATE METADATA`语句和`REFRESH`语句。




## 0x01 Hive元数据库

下图是我把hive元数据库倒出来之后，整理出来的表结构，整个元数据库表十分多，我只截取了一部分我认为相对来说比较重要的几个。

![](http://obg1rl2km.bkt.clouddn.com/impala-2-hive-metadata.png)
Impala 在传统的 MySQL 或 PostgreSQL 数据库称为 Metastore 上保持其表定义，Hive 也在相同的数据库上保存此类型的数据。因此，Impala 可以访问由 Hive 定义或加载的表。

对于具有大量数据或多个分区的表，检索表内所有元数据可能会花费很长时间，在某些情况下需要几分钟。因此，每个 Impala 节点缓存所有这些数据，以便在未来对同一表进行查询时重复使用。

如果更新表定义或表数据，集群中所有其他Impala Daemon必须接收到最新的元数据、更换过时的缓存元数据，然后再对该表发出查询。

## 0x02 INVALIDATE METADATA 语句

**`INVALIDATE METADATA`翻译成中文就是“作废元数据”的意思。**

**什么时候需要将一个或所有表的元数据标记为过时，即执行INVALIDATE METADATA？**

主要是下面这三种情况：

- 元数据发生更改。
- 从集群中的另一个 impalad 实例或通过 Hive 所做的更改。
- 与 Impala shell 或 ODBC 等客户端直接连接的数据库所做的更改。

在默认情况下`INVALIDATE METADATA`操作会刷新所有表的缓存元数据。与由 `REFRESH` 语句执行的元数据增量更新相比`INVALIDATE METADATA`是一项需要耗费大量资源的操作。如果指定了一个表名称，`INVALIDATE METADATA`只刷新用于该表的元数据。但是！即使对于单个表，INVALIDATE METADATA 也要比 REFRESH 耗费更多资源，因此当为现有表添加新数据文件这一常规情况下，应优先考虑REFRESH。

**注意：**

- 在当前节点进行ALTER TABLE、INSERT 或其它表修改语句后，该节点不需要更新impala元数据即可访问最新的数据。

- `-r`参数，有能需要为拥有多个分区的大型表耗费更多资源，因此在生产环境下的日常操作中尽量避免使用此选项。

## 0x03 REFRESH 语句

必须加表名参数。

`REFRESH table_name` 只适用于 Impala 已识别的表

要准确地对查询做出响应，充当协调员的 Impala 节点（通过 impala-shell、JDBC 或 ODBC 连接的节点）必须具有有关 Impala 查询中引用的数据库和表的当前元数据。

在 Impala (而不是 Hive) 中运行 ALTER TABLE、INSERT 或其他修改表的语句之后，Impala 节点的元数据更新不是必须的。Impala 通过目录服务自动处理元数据同步。

**注意：**在 Impala 1.2 及更高版本中，-r 的需要频率更低，因为 Impala 中的 SQL 语句导致的元数据更改会自动广播到所有Impala节点。


参照：

http://www.cloudera.com/documentation/archive/impala/2-x/2-1-x/topics/impala_invalidate_metadata.html

http://www.cloudera.com/documentation/archive/impala/2-x/2-0-x/topics/impala_refresh.html


***
2016-03-13 19:08:00 hzct
