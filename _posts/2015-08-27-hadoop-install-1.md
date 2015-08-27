---
layout: post
author: zhao
title:  "Hadoop：集群安装之一（安装环境说明和基本环境安装）"
date:   2015-08-27 22:14:54
categories: Hadoop
---

* content
{:toc}

##一、安装环境

###服务器信息

共四台阿里云主机，双网卡，双硬盘，有root权限。

~~~java

[root@z1 ~]#vim /etc/hosts

*.*.*.1    z1
*.*.*.2    z2
*.*.*.3    z3
*.*.*.4    z4

[root@z1 ~]# uname -a
Linux z1 2.6.32-431.23.3.el6.x86_64 #1 SMP Thu Jul 31 17:20:51 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

[root@z1 ~]# lsb_release -a
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch
Distributor ID:	CentOS
Description:	CentOS release 6.5 (Final)
Release:	6.5
Codename:	Final

[root@z1 ~]# df -lh
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  4.1G   15G  22% /
tmpfs           938M   16K  938M   1% /dev/shm
/dev/xvdb1       19G  558M   17G   4% /mnt

~~~

###软件版本

- CentOS: 6.5 (Final)
- Java: 1.7.0_79
- Hadoop: 2.7.1

###安装目录树

~~~
	mnt
	└── zhao
		├── data
		├── package
		└── soft
			└── hadoop
			└── jdk
			└── zookeeper
		
~~~


##二、基本环境部署

###新建Hadoop用户（root用户）

~~~java

[root@z2 ~]# useradd hadoop
[root@z2 ~]# passwd hadoop
Changing password for user hadoop.
New password: 
BAD PASSWORD: it is based on a dictionary word
BAD PASSWORD: is too simple
Retype new password: 
passwd: all authentication tokens updated successfully.

~~~

###配置ssh免密码登录（root+hadoop用户）

免密码登录不需要所有的机器都能互相免密码登录，只需一台服务器能够免密码登录其它服务器即可。

####修改/etc/ssh/sshd_config配置文件（root用户）

将以下三行去掉注释

~~~

RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys

~~~

然后重启服务

~~~

[root@z1 soft]# service sshd restart
Stopping sshd:                                             [  OK  ]
Starting sshd:                                             [  OK  ]

~~~

####本机免密码登录测试（hadoop用户）

下面操作在所有的四台服务器上执行

~~~java

[hadoop@z1 ~]$ ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_dsa):  //回车
Created directory '/home/hadoop/.ssh'.
Enter passphrase (empty for no passphrase): 	//回车
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_dsa.
Your public key has been saved in /home/hadoop/.ssh/id_dsa.pub.
The key fingerprint is:
2e:be:ba:df:71:42:da:db:52:b8:31:22:f1:ae:e7:8d hadoop@z1
The key's randomart image is:
+--[ DSA 1024]----+
|                 |
|                 |
|                 |
|    .            |
|     o  S.       |
|    . o=+ .      |
|     oo.==.      |
|     .o=o*       |
|    +BE.+..      |
+-----------------+

[hadoop@z1 ~]$ cd .ssh/
[hadoop@z1 .ssh]$ ls
id_dsa  id_dsa.pub
[hadoop@z1 .ssh]$ cat id_dsa.pub  >>  authorized_keys
[hadoop@z1 .ssh]$ chmod 600 authorized_keys
~~~

本机免密码登录测试

~~~

[hadoop@z1 .ssh]$ ssh localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
RSA key fingerprint is 36:61:03:1d:60:d3:89:e9:27:56:8e:3b:16:ec:89:7e.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
Last login: Thu Aug 27 14:02:33 2015 from ****

Welcome to aliyun Elastic Compute Service!

~~~

####远程免密码登录（hadoop用户）

将所有服务器上的authorized_keys，添加到z1的authorized_keys中，然后将该authorized_keys文件分发给集群中的其它服务器即可。

~~~
[hadoop@z1 .ssh]$ scp authorized_keys hadoop@z2:/home/hadoop/.ssh/
hadoop@z2's password: 
authorized_keys                                                           100% 2396     2.3KB/s   00:00    
~~~

然后远程免密码登录测试

~~~

[hadoop@z1 .ssh]$ ssh z2
Last login: Thu Aug 27 15:50:09 2015 from ****

Welcome to aliyun Elastic Compute Service!

~~~

###JDK环境安装（hadoo用户）

将jdk-7u79-linux-x64.tar.gz解压后移动到/mnt/zhao/soft中

~~~
[hadoop@z1 package]$ tar zxvf jdk-7u79-linux-x64.tar.gz
[hadoop@z1 package]$ mv jdk1.7.0_79/ ../soft/jdk
~~~

然后配置jdk环境变量

在/etc/profile中添加如下环境变量

~~~
export JAVA_HOME=/mnt/zhao/soft/jdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
~~~

然后输入java、javac、java -version验证。

~~~
[hadoop@z1 ~]$ vim /etc/profile
[hadoop@z1 ~]$ source /etc/profile 
[hadoop@z1 ~]$ java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
~~~

















