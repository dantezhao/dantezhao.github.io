---
layout: post
title:  "Mysql主从搭建"
categories: Database
tags: mysql
---

* content
{:toc}

## 前言

安装mysql的主从数据库。




## 准备工作

### 环境

- 两台Ubuntu16.04虚拟机，64位安装
- 两台虚拟机都是用apt安装的mysql，版本5.7.12
- 都安装开启了ssh服务
- 都关闭了防火墙
- 主节点：mysql001，从节点mysql002

## 搭建过程


### 主节点mysql001：

配置文件

```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```

#主从复制配置  
innodb_flush_log_at_trx_commit=1
sync_binlog=1

#需要备份的数据库
binlog_do_db           = test1
#不需要备份的数据库
binlog_ignore_db       = mysql
#启动二进制文件
log_bin                        = /var/log/mysql/mysql-bin.log
#服务器ID
server-id              = 1

```


重启服务

```
service mysql restart
```

创建用户并分配权限

```
mysql> create user 'dante'@'mysql002' identified by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to 'dante'@'mysql002' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      698 | test1        | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```


### 从节点mysql002：

配置文件

```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
#服务器ID
server-id              = 2
```

重启服务
```
service mysql restart
```

设置连接信息

```
mysql> change master to master_host='mysql001',
    -> master_user='dante',
    -> master_password='123456',
    -> master_log_file='mysql-bin.000001',
    -> master_port=3306,
    -> master_log_pos=2005,
    -> master_connect_retry=10;
Query OK, 0 rows affected, 2 warnings (0.04 sec)

```

启动slave

```
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)
```

查看连接信息

```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql001
                  Master_User: dante
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 154
               Relay_Log_File: dante-VirtualBox-relay-bin.000003
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 751
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 7a04fb5e-3db5-11e6-abd3-08002703bba1
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

这个时候就能看到同步过来的表了。


## 错误总结

整个过程就遇到了一个问题，就是连接主库的问题，一直连接不上，这种情况首先考虑的就是防火墙的问题。由于是16.04，我用ufw关闭了防火墙，打开了3306端口。



```
# nmap -p 22 mysql001

Starting Nmap 7.01 ( https://nmap.org ) at 2016-06-29 14:22 CST
Nmap scan report for mysql001 (10.1.196.96)
Host is up (0.00045s latency).
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:03:BB:A1 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds

```

用nmap扫也没问题了，但是还是连不上。

```
 Slave_IO_Running: Connecting
```

查看一下mysql的日志，也报错为连接失败

```
2016-06-29T06:22:45.985444Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 19, Error_code: 2003
2016-06-29T06:22:55.986779Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 20, Error_code: 2003
2016-06-29T06:23:05.987615Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 21, Error_code: 2003
2016-06-29T06:23:15.988919Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 22, Error_code: 2003
2016-06-29T06:23:25.990153Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 23, Error_code: 2003
2016-06-29T06:23:35.991296Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 24, Error_code: 2003
2016-06-29T06:23:45.992279Z 10 [ERROR] Slave I/O for channel '': error connecting to master 'dante@mysql001:3306' - retry-time: 10  retries: 25, Error_code: 2003

```
这就是典型的连接失败。这时候我想起来是不是和绑定的地址有关，再看一下mysql的配置文件，这次改了一个参数。

```
#bind-address           = 127.0.0.1
bind-address           = mysql001
```

这次就一切OK了。



## 补充

排查问题的时候参照了这哥们的记录，虽说最终解决方式没用上，但是感觉有必要记录一下。

[博客地址](http://blog.itpub.net/27099995/viewspace-1294075)

导致slave_IO_Running 为connecting 的原因主要有以下 3 个方面：  

- 1、网络不通  
- 2、密码不对  
- 3、pos不对

解决步骤：

- 1、对于第一个问题，一般情况下都是可以排除的，也是最容易排除的。
- 2、在主库上修改用来复制的用户的密码。
- 3、 在做chang to 的时候注意log_pos 是否跟此时主机的一样。在主机上 show master status \G ;可以查看到。

## 总结


整个过程花费了一个多小时，本来感觉半个多小时就搞定了呢，结果出了一个端口连接的问题，解决这么久......
以后要加强。

******
2016-06-29 hzct
