---
layout: post
title:  "Mysql查看数据的的物理大小的两种方法"
categories: Database
tags:  mysql sql
---

* content
{:toc}


## 前言

经常用mysql，发现居然从来不知道mysql的表大小。




## 方法一

### information_schema 数据库

information_schema存放了不少有用的信息。

```
mysql> use information_schema
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
```

### TABLES表

这次主要关注TABLES表。

```
mysql> show tables;
+---------------------------------------+
| Tables_in_information_schema          |
+---------------------------------------+
| SCHEMATA                              |
| SCHEMA_PRIVILEGES                     |
| ......                       			|
| SESSION_VARIABLES                     |
| STATISTICS                            |
| TABLES                                |
| VIEWS                                 |
+---------------------------------------+
```

### 查看表大小

一个表的总大小应该是和视图索引都相关，因为我要看的表只是单纯放了些数据，就只查看表大小了。

```
mysql> select DATA_LENGTH, TABLE_NAME from TABLES;
+-------------+---------------------------------------+
| DATA_LENGTH | TABLE_NAME                            |
+-------------+---------------------------------------+
|  1238368256 | tablename                     		  |
+-------------+---------------------------------------+
53 rows in set (0.01 sec)
```

## 方法二

### 配置文件
查找配置文件，windows的是my.ini。我在linux上面查找。

```
[root@mysql1 /tmp/dante]# find / -name my.cnf
/etc/my.cnf

```

查看文件，可以看到具体的信息。

```
vim /etc/my.cnf

datadir = /data/mysqldb/
# port = .....
socket = /data/mysqldb/mysql.sock
```

### 物理位置

查找到物理位置之后就可以看到，在物理位置中，每个数据库是一个目录，我的数据库目录是dbname。

```
[root@mysql1 /data/mysqldb]# ll -h
总用量 1.4G
drwx------ 2 mysql mysql 4.0K 4月   1 20:26 dbname
-rw-rw---- 1 mysql mysql 1.4G 4月   1 19:52 ibdata1
-rw-rw---- 1 mysql mysql 5.0M 4月   1 19:52 ib_logfile0
-rw-rw---- 1 mysql mysql 5.0M 4月   1 19:52 ib_logfile1
drwx------ 2 mysql mysql 4.0K 10月 17 10:49 mysql
-rw-r----- 1 mysql root   642 11月 10 14:37 mysql1.err
-rw-rw---- 1 mysql mysql 1.1K 4月   1 19:21 mysql-bin.000004
-rw-rw---- 1 mysql mysql   19 11月 10 14:37 mysql-bin.index
srwxrwxrwx 1 mysql mysql    0 11月 10 14:37 mysql.sock
-rw-r----- 1 mysql root  1.8K 11月  2 23:51 test-46.err
```

进入dbname目录之后，就会发现每张表都是一个文件，但是非常小，说明数据应该是在外面存着的，也就是那个ibdata1。

```
[root@mysql1 /data/mysqldb/dante]# ll -h
-rw-rw---- 1 mysql mysql  61 4月   1 17:19 db.opt
-rw-rw---- 1 mysql mysql 18K 4月   1 18:38 tablename.frm

```
来源：
http://blog.csdn.net/zhaodedong
http://zhaodedong.leanote.com
http://zhaodedong.com

******
2016-04-01 20:35:00 hzct
******
