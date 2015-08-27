---
layout: post
author: zhao
title: "Linux：集群工具ClusterShell"
date:   2015-04-16 22:51:50
categories: Linux
---

* content
{:toc}


##简介

实验室机房有大概百台的服务器需要管理，加上需要搭建Hadoop以及Spark集群等，因此，一个轻量级的集群管理软件就显得非常有必要了。经过一段时间的了解以及尝试，最终选择了clustershell这个软件，原因如下。

- 安装方便。一条指令就能轻松安装。

- 配置方便。很多集群管理软件都需要在所有的服务器上都安装软件，而且还要进行很多的连接操作。clustershell就相当的方便了，仅仅需要所有机器能够ssh无密码登录即可，然后只在一台服务器上安装clustershell即可。（这一点正好和Hadoop集群结合起来使用，相当方便）

- 使用方便。clustershell的命令相对来说非常简单，只有一两个指令以及三四个参数需要记。

##安装

###安装clustershell

安装非常简单，只有一条指令即可，一般服务器都是红帽系列的，使用yum安装。

~~~
yum install clustershell
~~~

###配置ssh无密码登录

配置ssh登录相对比较简单，在搭建hadoop集群的时候都会需要这一步。

###配置/etc/hosts

在hosts中文件中将ip和主机名对应起来，使用比较方便。


