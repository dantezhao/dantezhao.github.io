---
layout: post
title:  "Mysql 使用load data导入数据从库同步测试"
categories: 数据为王
tags: Mysql
---

* content
{:toc}

## 前言

有个业务，需要从impala中查询数据后，load到mysql中，但是听说load的数据不会同步到从库中，因此自己做个测试。




## 测试

### 准备

建表语句：

```
CREATE TABLE test ( id int not null primary key,name char(20) );
```

数据：

```
1,name
```

### 数据操作

主节点：

```
mysql> LOAD DATA INFILE '/var/lib/mysql-files/data.txt' INTO TABLE test1.test FIELDS TERMINATED BY ',';
Query OK, 1 row affected (0.01 sec)
Records: 1  Deleted: 0  Skipped: 0  Warnings: 0

mysql> select * from test;
+----+------+
| id | name |
+----+------+
|  1 | name |
+----+------+
1 row in set (0.00 sec)

```
从节点：

```
mysql>  select * from test;
+----+------+
| id | name |
+----+------+
|  1 | name |
+----+------+
1 row in set (0.00 sec)

```

观测到数据已经过来了。

## 排查错误。

报错不能使用load，估计是新mysql的问题。

```
mysql> LOAD DATA INFILE '/root/data.txt' INTO TABLE test1.test FIELDS TERMINATED BY ',';
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

```

找到官网的解释。
```
This variable is used to limit the effect of data import and export operations, such as those performed by the LOAD DATA and SELECT ... INTO OUTFILE statements and the LOAD_FILE() function. These operations are permitted only to users who have the FILE privilege.

secure_file_priv may be set as follows:

If empty, the variable has no effect.

If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server will not create it.

If set to NULL, the server disables import and export operations. This value is permitted as of MySQL 5.7.6.
```

看一下变量，果真设置了哦。
```
mysql> SHOW VARIABLES
......
| secure_file_priv                                         | /var/lib/mysql-files/
......
```
但是发现不让自己修改，这是几个意思。
···
mysql> set secure_file_priv=no;
ERROR 1238 (HY000): Variable 'secure_file_priv' is a read only variable
···

先用个简单的方法凑合一下，按照官网的意思，我只要是在那个指定目录中操作就行了。然后果真行了。

## 总结

目前使用的版本，load数据后，会同步到从库。

年轻多折腾。

***
2016-06-29 hzct
