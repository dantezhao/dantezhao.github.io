---
author: zhao
title:  Linux：集群工具ClusterShell
date:   2014-10-23
categories: linux
tags:
	- cluster
	- shell
description: 实验室机房有大概百台的服务器需要管理，加上需要搭建Hadoop以及Spark集群等，因此，一个轻量级的集群管理软件就显得非常有必要了。经过一段时间的了解以及尝试，最终选择了clustershell这个软件。
---

## 简介

实验室机房有大概百台的服务器需要管理，加上需要搭建Hadoop以及Spark集群等，因此，一个轻量级的集群管理软件就显得非常有必要了。经过一段时间的了解以及尝试，最终选择了clustershell这个软件，原因如下。

- 安装方便。一条指令就能轻松安装。

- 配置方便。很多集群管理软件都需要在所有的服务器上都安装软件，而且还要进行很多的连接操作。clustershell就相当的方便了，仅仅需要所有机器能够ssh无密码登录即可，然后只在一台服务器上安装clustershell即可。（这一点正好和Hadoop集群结合起来使用，相当方便）

- 使用方便。clustershell的命令相对来说非常简单，只有一两个指令以及三四个参数需要记。

## 安装

### 安装clustershell

安装非常简单，只有一条指令即可，一般服务器都是红帽系列的，使用yum安装。

```
yum install clustershell
```

### 配置ssh无密码登录

配置ssh登录相对比较简单，在搭建hadoop集群的时候都会需要这一步。

### 配置/etc/hosts

在hosts中文件中将ip和主机名对应起来，使用比较方便。

### 配置关键文件

clustershell的配置文件在`/etc/clustershell`目录下，其中的groups是最常用的，我只配置了这一个文件。

`all: z[1-4]`是指所有的节点，在使用的是通过`-a`来选择all

`hadoop: z[1-4]`，是指定hadoop组中有四个节点，分别是z1，z2，z3，z4。其它的配置也类似，可以加入多个组，使用的时候通过`-g hadoop`来选择。

```
adm: example0
oss: example4 example5
mds: example6
io: example[4-6]
compute: example[32-159]
gpu: example[156-159]
all: z[1-4]

hadoop: z[1-4]
```

### 使用

clustershell在使用的时候有一个非常重要的指令就是clush，目前为止我也只用到了这一个指令。

clush [-option] 后面就是日常的linux上执行的指令即可，没什么复杂的，都十分简单。

但是有一点要注意，clustershell执行的类似与一次操作的指令，比如你可以touch一个新文件在所有节点上，但是你不能同时在所有节点上vim编辑一个新文件。细节还需琢磨。

clush有几个比较重要的参数：

``` 
-b : 相同输出结果合并
-w : 指定节点
-a : 所有节点
-g : 指定组
--copy : 群发文件
```

#### 输出所有节点的JAVA_HOME信息

```
[hadoop@z1 ~]$ clush -b -a echo $JAVA_HOME
---------------
z[1-4] (4)
---------------
/mnt/zhao/soft/jdk
```

#### 删除指定节点的文件

删除 z2,z3,z4三个节点上的/mnt/zhao/soft/jdk文件夹

```
 clush -w z2,z3,z4 rm -rf /mnt/zhao/soft/jdk
```


#### 集群分发文件

把本地的一个/mnt/zhao/package/jdk-7u79-linux-x64.tar.gz文件分发到hadoop组中所有节点的/mnt/zhao/package/目录下

```
[hadoop@z1 ~]$ clush -b -g hadoop --copy /mnt/zhao/package/jdk-7u79-linux-x64.tar.gz --dest /mnt/zhao/package/
```

#### 集群查看文件

查看所有hadoop组中/mnt/zhao/package/目录下的文件，输出结果合并。

```
[hadoop@z1 package]$ clush -b -g hadoop ls /mnt/zhao/package/
---------------
z[2-4] (3)
---------------
jdk-7u79-linux-x64.tar.gz
---------------
z1
---------------
clustershell-1.6.tar.gz
hadoop-2.7.1.tar.gz
jdk-7u79-linux-x64.tar.gz
```

#### 官方文档

clustershell还有很多功能，但是需求驱动学习，目前我能用到的功能在上面体现了，有需求的话会再学习深一点。

另外官方文档里面的额内容非常丰富，比一些教程什么的要实在的多，可以参照。

https://cloud.github.com/downloads/cea-hpc/clustershell/ClusterShell_UserGuide_EN_1.6.pdf