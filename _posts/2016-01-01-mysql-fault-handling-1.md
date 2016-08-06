---
layout: post
title:  "Mysql：一次mysql错误处理"
categories: Database
tags:  mysql
---

* content
{:toc}

## 前言

在腾讯云上了搞一套lamp的环境，正在测试，结果被某个无良的孩子通过phpadmin把mysql的各个权限都关了，直接重装mysql感觉太掉身价，因此开始了折腾之路。





## 错误现象

**root没有select权限了。**

```
mysql> select * from typecho_users;
ERROR 1142 (42000): SELECT command denied to user 'root'@'localhost' for table 'typecho_users'
```

**也不能进行附权限操作了**

```
mysql> grant all on typecho.* to root@localhost identified by "mylove";
ERROR 1044 (42000): Access denied for user 'root'@'localhost' to database 'typecho'
```

## 处理过程

先查一下用户的权限。

```
mysql> use typecho
Database changed
mysql> show grants for 'root'@localhost;
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@localhost                                                                                                                                                                                                                             |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT RELOAD, SHUTDOWN, PROCESS, REFERENCES, SHOW DATABASES, SUPER, LOCK TABLES, REPLICATION SLAVE, REPLICATION CLIENT, CREATE USER ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*FAABED87BF077822B57AF4F181A657052372A0B6' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                                                                                                                                          |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

我就想了，mysql总该有一个专门记录权限的地方吧。
```
mysql> use mysql;
Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
28 rows in set (0.00 sec)
```

好嘛，也不能进，这个权限是被删除的多彻底啊 。

```
mysql> select * from user limit 10;
ERROR 1142 (42000): SELECT command denied to user 'root'@'localhost' for table 'user'
```

好bia。 `mysql --help`貌似没什么有用的帮助

然后不小心找到了一种方式，无权限登录模式！！！

```
/etc/init.d/mysqld stop
/etc/init.d/mysqld start --skip-grant-table

```


登录！

我擦，可以正常访问，也能各种select，但是不能使用附权限。


```

mysql> select * from typecho_users;
+-----+-------+----------------------------------+--------------------------+------------------------+------------+---------+------------+------------+---------------+----------------------------------+
| uid | name  | password                         | mail                     | url                    | screenName | created | activated  | logged     | group         | authCode                         |
+-----+-------+----------------------------------+--------------------------+------------------------+------------+---------+------------+------------+---------------+----------------------------------+
|   1 | admin | 23f474aef895fa9f10b9e5bb5ab804d5 | webmaster@yourdomain.com | http://www.typecho.org | admin      |       0 | 1455861175 | 1455858464 | administrator | 30467e3f5eb4f30eff282e2df2beebd5 |
+-----+-------+----------------------------------+--------------------------+------------------------+------------+---------+------------+------------+---------------+----------------------------------+
1 row in set (0.00 sec)

mysql>  grant all on typecho.* to root@localhost identified by "mylove";
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement

```

囧！

脑子抽筋了，感觉没方法处理了，先去工作了。

然后经妹子提醒，在无权限登录模式下，可以用update来修改root用户的权限，接下来就OK了。update的语句google可搜。


******
2016-01-01 22:14:54 于 hzct
