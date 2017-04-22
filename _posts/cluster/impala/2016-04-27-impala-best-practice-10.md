---
layout: post
title:  "Impala实践之十：impala最佳实践（转、译、整理）"
categories: 漫步云端
tags: Impala
---

* content
{:toc}

## 前言

最近在看impala原理时候翻出来的一些tip，帮助更好地使用impala，自己整理一下。




## 0x01 杂项

### 1. Impala 使用缓存吗？

Impala 不会缓存数据，但它缓存一些表和文件的元数据。尽管因为数据集被缓存到 OS 的缓冲区中，接下来的重复查询可能运行的更快，Impala 不会明确的控制这些。

## 0x02 Impala任务失败

### 1. 为什么 SELECT 查询会失败？

当一个 SELECT 语句失败了，原因通常是以下类别之一：

- 因为性能、容量、或网络问题影响了特定的节点导致的超时
- 连接查询的过多内存数用，这一查询的结果会自动取消
- 处理查询中特定的 WHERE 子句时，影响到每一节点上本地代码如何生成的底层问题。例如，特定节点上可能会生成它的处理器不支持的机器指令。假如日志中的错误信息猜测是无效指令(illegal instruction)，考虑临时关闭生成本地代码，并重试这个查询
- 异常的输入数据，例如包含一个巨大的长行的文本数据文件(a text data file with an enormously long line)，或者使用了没有在 CREATE TABLE 语句中 FIELDS TERMINATED BY 子句中设置的分隔符(or with a delimiter that does not match the character specified in the FIELDS TERMINATED BY clause of the CREATE TABLEstatement)

### 2. 为什么 INSERT 查询会失败？


当 INSERT 语句失败时，通常是因为超出 Hadoop 组件的一些限制，特别是 HDFS。

- 由于可能会在 HDFS 并发打开许多文件和关联的进程，插入到分区表的操作是一个费力(strenuous)操作。Impala 1.1.1 包含了一些改进，以更有效的分发工作，这样每个分区使用一个节点写入值，而不是没一个节点一个单独的数据文件
- INSERT 语句中 SELECT 部分的特定表达式会产生复杂的执行计划，并导致低效的 INSERT 操作。请尽量使源表和目标表中列的数据类型匹配，例如，如果必要，在源表上执行 ALTER TABLE ... REPLACE COLUMNS 语句。请尽量避免在 SELECT 位置使用 CASE 表达式，因为相比保持列不变或通过内置函数转换列，CASE 会导致结果更难预测
- 请做好准备提升你的 HDFS 配置设置中的一些限制，可以临时的在 INSERT 执行时，如果你频繁运行这些 INSERT 语句作为 ETL 管道的一部分，也可以永久修改
- 依赖于目标表的文件格式，INSERT 语句的资源使用可能会变化。插入到 Parquet 表是内存密集型操作，因为每一个分区的数据会缓存到内存里，直到它达到 1G，这时候数据文件才写入到硬盘。当执行 INSERT 语句时候，如果查询中源表的统计信息可用，Impala 可以更高效的分布工作。参见 How Impala Uses Statistics for Query Optimization 了解如何采集统计信息


## 0x03 Impala的内存

### 1. 当数据集超出可用内存时会发生什么？

目前来说，假如在某一节点上处理中间结果集所需的内存超出了这一节点上 Impala 可用的内存，查询会被取消。你可以调整每一节点上 Impala 的可用内存，也可以对你最大的查询微调连接策略来减少内存需求。我们计划在将来支持外部连接和排序。

但请记住，使用内存的大小并不是跟输入数据集的大小直接相关。对于聚合来说，使用的内存跟分组后的行数有关。对于连接来说，使用的内存与除了最大的表之外其他所有表的大小相关，并且 Impala 可以采用在每个节点之间拆分大的连接表而不是把整个表都传输到每个节点的连接策略。

### 2. 哪些是内存密集型操作？

假如查询失败，错误信息是 "memory limit exceeded"，你可能怀疑有内存泄露(memory leak)。其实问题可能是因为查询构造的方式导致 Impala 分配超出你预期的内存，从而在某些节点上超出 Impala 分配的内存限制(The problem could actually be a query that is structured in a way that causes Impala to allocate more memory than you expect, exceeded the memory allocated for Impala on a particular node)。一些特别内存密集型的查询和表结构如下：

使用动态分区的 INSERT 语句，插入到包含许多分区的表中(特别是使用 Parquet 格式的表，这些表中每一个分区的数据都保存到内存中，直到它达到 1 GB 并被写入到硬盘里)。考虑把这样的操作分散成几个不同的 INSERT 语句，例如一次只加载一年的数据而不是一次加载所有年份的数据
在唯一或高基数(high-cardinality)列上的 GROUP BY 操作。Impala 为 GROUP BY 查询中每一个不同的值分配一些处理结构(handler structures)。成千上万不同的 GROUP BY 值可能超出内存限制
查询涉及到非常宽、包含上千个列的表，特别是包含许多 STRING 列的表。因为 Impala 允许 STRING 值最大不超过 32 KB，这些查询的中间结果集可能需要大量的内存分配

### 3.何时 Impala 分配(hold on to)或释放(return)内存？

Impala 使用 tcmalloc 分配内存，一款专为高并发优化的内存分频器。一当 Impala 分配了内存，它保留这些内存用于将来的查询。因此，空闲时显示 Impala 有很高的内存使用是很正常的。假如 Impala 检测到它将超过内存限制(通过 -mem_limit 启动选项或 MEM_LIMIT 查询选项定义)，它将释放当前查询不需要的所有内存。

**注意：** 当通过 JDBC 或 ODBC 接口执行查询，请确保在之后调用对应的关闭方法。否则，查询关联的一些内存不会释放。


## 0x04 Impala使用场景

### 1. 什么情况下适合使用 Impala 而不适合 Hive 和 MapReduce？

Impala 非常适合在大的数据集上，为交互式探索分析执行 SQL。Hive 和 MapReduce 则适合长时间运行的、批处理的任务，例如 ETL。

### 2. Impala 是否需要 MapReduce ？如果 MapReduce 停了，Impala 是否能正常工作？

Impala 用不到 MapReduce。

### 3. Impala 是否可以用于复杂事件处理？

例如，在工业环境中，许多客户端可能产生大量的数据。Impala 是否可用与分析这些数据，发现环境中显著的变化？

复杂事件处理(Complex Event Processing,CEP) 通常使用专门的流处理系统处理。Impala 不是流处理系统，它其实更像关系数据库。

## 0x05 Impala和hive互操作

### 1. impala和hive如何读取对方修改过的数据？

Impala在读取hive修改过后的元数据后，需要进行`invalidate metadata`操作，具体的使用说明请参考我的其它博客。

### 2. Impala 中的所有查询都可以在 Hive 中执行吗？

是的。尽管在一些查询如何处理方面有细微的差别，但是 Impala 查询也可以在 Hive 中完成。Impala SQL 是 HiveQL 的子集，有一些功能限制如变换(transforms)。关于具体的 Impala SQL 方言。关于 Impala 内置函数，参见 Built-in Functions。关于不支持的 HiveQL 特性，详见 [SQL Differences Between Impala and Hive](http://www.cloudera.com/documentation/archive/impala/2-x/2-1-x/topics/impala_langref_unsupported.html#langref_unsupported)。

**注意：** hive和impala有一些窗函数的使用方法是不一样的，比如impala里面有`group_count`，但是hive里面没有，取而代之的是`collect_list`和`collect_set`。

### 3. 可以用 Impala 查询已经在 Hive 和 HBase 加载的数据吗？

允许 Impala 查询 Hive 管理的表，不管它是存放在 HDFS 还是 HBase中，都不需要额外的步骤。请确保已经正确的配置 Impala 访问 Hive metastore，并且你准备好了。请记住，默认的 impalad 使用 impala 用户运行，所以你可能需要调整一些文件的权限，这取决于你目前权限多么严格。

### 4. Impala 是否需要 Hive？

Hive metastore 服务是必需的。Impala 与 Hive 共享同一个 metastore 数据库，透明的允许 Impala 和 Hive 访问相同的表。

Hive 本身是可选的，并且不需要跟 Impala 安装在同一个节点上。相比目前 Impala 支持的写(插入)操作(的文件格式)，Impala 支持更多类型的读取(查询)操作；对于使用的特定的文件格式，你应当使用 Hive 向表里插入数据。参见 How Impala Works with Hadoop File Formats 了解详细信息。

***
2016-04-27 10:59:00 hzct
