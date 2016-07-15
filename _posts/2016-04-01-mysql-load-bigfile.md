---
layout: post
title:  "Mysql大文件导入方法以及性能对比"
categories: Mysql
tags:  mysql sql database
---

* content
{:toc}

## 前言

### 背景
今天被妹子问到一个问题，往mysql中导入1000W条数据，哪种方式比较快，我居然被问到了，说实话我还真心不知道。

那就找数据呗，搞数据研发的好处就是，数据肯定是很多的，正好还有测试集群。开始在线上找数据。

由于我们很多mysql的表会通过sqoop抽到hdfs中，所有就直接在hdfs中把数据提出来了，不给线上mysql什么压力了。





### 对比实验

现在使用两种方式分别往mysql数据库中插入数据。
- 一种是load data的形式，
- 一种使用sqoop从hdfs中抽。

### 数据

- 200W+条数据
- 12个字段，int居多
- 1.2G
- ENGINE=InnoDB


### 测试环境

测试时，集群本身没什么数据，也没有任何任务，基本可以看成纯净的测试环境。

- 5台hadoop集群，标准8G内存，1T硬盘，cdh5.4.7，各种组件齐全。
- 1台单独mysql服务器。

用两种测试方法，分别使用独自的新表，互不影响。但是是在同一台mysql中。



## MySQL load data

使用mysql自带的load工具


```
LOAD DATA INFILE '/tmp/dante/part-m-00000' INTO TABLE dante.tablename FIELDS TERMINATED BY ',';
```

### 结果

执行完成总共用了14分钟，感觉还能不错哦。

```
Query OK, 2106860 rows affected, 65535 warnings (14 min 7.19 sec)
Records: 2106860  Deleted: 0  Skipped: 0  Warnings: 4213720
```

### 遇到的错误信息

#### 错误一

我是直接再hdfs中拿的数据，数据就是通过sqoop从mysql抽出来的，结果还是报了这个错误

```
ERROR 1406 (22001): Data too long for column 'is_receive' at row 1
```

其实这次此时我不太关注什么数据类型，那就把出问题的数据类型改成varchar。

#### 错误二

```
ERROR 1261 (01000): Row 1 does not contain data for all columns
```

在 MySQL 中使用 load data infile 命令导入数据文件到 MySQL 数据库中的时候，如果遇到 MySQL 错误：“ERROR 1261 (01000)”，则很可能是由于数据文件中的列数跟 MySQL 数据表字段数目没有完全匹配，并且 sql_mode 设为 strict 模式的缘故。要想在这种情况下继续导入数据到 MySQL 表中，则需要设置 MySQL sql_mode 变量。

```
mysql> show variables like 'sql_mode';
+---------------+--------------------------------------------+
| Variable_name | Value                                      |
+---------------+--------------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |

```

把它改成了ANSI就行了。

```
set @@sql_mode=ANSI; 
```

## sqoop

从sqoop导入到mysql之前已经把数据放在了hdfs的`/tmp/dante/test`中。

```
sqoop-export --connect jdbc:mysql://mysql:3306/dante --username root --password root --table tablename --export-dir /tmp/dante/test --input-fields-terminated-by ',' 
```

### 结果

sqoop明显要比mysql自身的导入方式要多了，六七分钟就完成了导入。

### 错误

#### 错误一


```
16/04/01 19:17:23 ERROR manager.SqlManager: Error executing statement: java.sql.SQLException: null,  message from server: "Host 'test' is not allowed to connect to this MySQL server"
java.sql.SQLException: null,  message from server: "Host 'test' is not allowed to connect to this MySQL server"
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:959)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:898)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:887)
	at com.mysql.jdbc.MysqlIO.doHandshake(MysqlIO.java:1038)
	at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2254)
	at com.mysql.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:2285)
	at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2084)
	at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:795)
	at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:44)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)
	at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:400)
	at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:327)
	at java.sql.DriverManager.getConnection(DriverManager.java:664)
	at java.sql.DriverManager.getConnection(DriverManager.java:247)
	at org.apache.sqoop.manager.SqlManager.makeConnection(SqlManager.java:880)
	at org.apache.sqoop.manager.GenericJdbcManager.getConnection(GenericJdbcManager.java:52)
	at org.apache.sqoop.manager.SqlManager.execute(SqlManager.java:739)
	at org.apache.sqoop.manager.SqlManager.execute(SqlManager.java:762)
	at org.apache.sqoop.manager.SqlManager.getColumnInfoForRawQuery(SqlManager.java:270)
	at org.apache.sqoop.manager.SqlManager.getColumnTypesForRawQuery(SqlManager.java:241)
	at org.apache.sqoop.manager.SqlManager.getColumnTypes(SqlManager.java:227)
	at org.apache.sqoop.manager.ConnManager.getColumnTypes(ConnManager.java:295)
	at org.apache.sqoop.orm.ClassWriter.getColumnTypes(ClassWriter.java:1833)
	at org.apache.sqoop.orm.ClassWriter.generate(ClassWriter.java:1645)
	at org.apache.sqoop.tool.CodeGenTool.generateORM(CodeGenTool.java:107)
	at org.apache.sqoop.tool.ExportTool.exportTable(ExportTool.java:64)
	at org.apache.sqoop.tool.ExportTool.run(ExportTool.java:100)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:143)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:179)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:218)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:227)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:236)
16/04/01 19:17:23 ERROR tool.ExportTool: Encountered IOException running export job: java.io.IOException: No columns to generate for ClassWriter
```

这个原因就是，mysql的权限没有设置好，集群中的所有机器都要对mysql有访问权限才行。


## 分析

其实个人感觉，这两种方式没有太大的可比性，因为有好几个条件是不同的，比如说使用load导入的时候，文件是在本地，没有网络传输的损耗，但是使用sqoop的时候需要从各个机器传输。而且使用sqoop的时候相当于是一个集群再向mysql导入数据。这些都会对最后的结果有很大的影响。还有一个就是，测试的数据量也没什么参考价值，毕竟只是自己试着玩玩的。

不过在进行方案选取的时候，使用这些测试还是有一些参考价值的，有时候不管原因是什么，最后某种方案的效果比较好，其实是可以考虑直接选用的。

******
2016-04-01 19:00:00 hzct



















