---
layout: post
title:  "DRBD Centos6.5（64bit）编译安装，双主模式"
categories: 不折腾被变羊
tags:  Linux
---

* content
{:toc}

## 一、简介

DRBD(Distributed Rep;icate Block Device)是基于块设备在不同的高可用服务器之间同步和镜像数据的软件，通过它可实现在网络中的两台服务器之间基于块设备级别的实时或者异步镜像或同步复制。

**DREB可以理解为网络的raid1。**

Raid1：其原理就是将一块硬盘的数据以相同位置指向另一块硬盘的位置。RAID 1的操作方式是把用户写入硬盘的数据百分之百地自动复制到另外一个硬盘上。由于对存储的数据进行百分之百的备份，在所有RAID级别中，RAID 1提供最高的数据安全保障。RAID1是将一个两块硬盘所构成RAID磁盘阵列，其容量仅等于一块硬盘的容量，因为另一块只是当作数据“镜像”。
块设备可以是磁盘分区、LVM逻辑卷、整块磁盘。





## 二、单主模式和双主模式

单主模式：一个集群内一个资源在任何给定的时间内仅有一个primary角色，另一个为secondary。文件系统可以是ext3、ext4、xfs等。

单主模式下，在primary节点上有如下操作：格式化： mkfs.ext4 /dev/drbd1，然后将/dev/drbd1挂载到之前创建的/db目录。在secondary节点，启动drbd后是无法挂载/dev/sda6。
备份的过程是，先在primary上的/db目录中写入数据，然后关闭secondary的drbd服务：drbdadm down r0; 之后就可以把secondary的/dev/sda6挂载到目录/db下，就可以看到primary中的数据了。

双主模式：对于一个资源，在任何给定的时刻该集群都有两个primary节点，也就是drbd两个节点均为primary，因此可以实现并发访问。此时的文件系统应该是GFS或者OCFS2。

在双主模式，需要在配置文件/global-common.conf中加入allow-two-primaries yes;一项，在两个节点上同时执行 `drbdadm primary –force r0`，将两个节点提升为primary节点。然后在一台节点上执行：
`mkfs.gfs2 -j 2 -p lock_dlm -t gfscluster:gfslocktable /dev/drbd1`
将其格式化为gfs文件系统。然后在两个节点上执行`mount /dev/drbd1 /db`，即可实现双主模式
在双主模式下，传入4G左右的文件成功，删除文件成功，两台节点能保证数据一致，而且基本同步。比如从本地传入greencloud1一个4G的文件，greencloud1中传入1G的话，在greencloud2也会同步过去1G的大小。

## 三、安装过程

操作系统版本为Centos6.5，drbd软件版本8.4.1

### 0.服务器准备

#### 0.1.分区设置

暂定的安装方式是：在安装系统的时候，留出一定大小的磁盘空间不做分配，为后续安装drbd使用。

#### 0.2. 关闭防火墙

```
/etc/init.d/iptables stop
chkconfig iptables off
```

#### 0.3. 修改ip和hostname对应

```
vi /etc/hosts
```

在配置drbd的时候需要用到主机名，因此添加ip和hostname对应关系

#### 0.4.双机互相通信

```
greencloud1:
ssh-keygen -t rsa -P ''
greencloud2:
ssh-copy-id -i .ssh/id_rsa.pub root@greencloud1
```

### 1.安装所需依赖（以下步骤在两台服务器上共同执行）

#### 1.1.gcc和内核

```
yum -y install gcc kernel-devel kernel-headers flex
```

#### 1.2.gfs和cman

```
yum install gfs2-utils.x86_64 cman.x86_64
```
双主模式需要使用分布式文件方案

GFS是一个可扩展的分布式文件系统，用于大型的、分布式的、对大量数据进行访问的应用。

Cluster manager 简称CMAN，是一个分布式集群管理工具，运行在集群的各个节点上，为RHCS提供集群管理任务。

### 2.安装drbd

#### 2.0.磁盘分区

使用fdisk出一个sda6分区，分区步骤如图，也可google一下更详细的分区介绍。

分区完成后建议重启计算机，因为执行partprobe后还是在安装drbd的时候会报错，重启是最保险的做法。

#### 2.1.下载

将tar压缩包放入某文件夹内

```
tar xzf drbd-8.4.1.tar.gz
cd drbd-8.4.1
./configure --prefix=/usr/local/drbd --with-km    //编译并指定目录，-km不清楚作用
make KDIR=/usr/src/kernels/2.6.32-431.20.5.el6.x86_64/
make install  
mkdir -p /usr/local/drbd/var/run/drbd  
cp /usr/local/drbd/etc/rc.d/init.d/drbd /etc/rc.d/init.d  
chkconfig --add drbd   //chkconfig --add“ 只是设置一个存在的service为自动启动
chkconfig drbd on
```

#### 2.2.安装drbd模块

```
cd drbd
make clean
make KDIR=/usr/src/kernels/2.6.32-431.20.5.el6.x86_64/
cp drbd.ko /lib/modules/`uname -r`/kernel/lib/
depmod   //分析可载入模块的相依性
```

#### 2.3.配置drbd文件

```
mkdir /db   /用于备份目录
```

#### 2.3.1.配置drbd.conf文件

```
vi /usr/local/drbd/etc/drbd.conf
写入
include "drbd.d/global_common.conf";
include "drbd.d/*.res";
```

#### 2.3.2.配置global_common.conf文件

```
vi /usr/local/drbd/etc/drbd.d/global_common.conf
设置以下内容：
global {
       usage-count yes;
}
common {
       startup {
              become-primary-on both;     
}
       disk {
              fencing resource-and-stonith;
       }
       net {
              protocol C;
              allow-two-primaries yes;   //设置为双主模式
              after-sb-0pri discard-zero-changes;
              after-sb-1pri discard-secondary;
              after-sb-2pri disconnect;
       }
}
```

#### 2.3.3.配置r0.res文件

```
vi /usr/local/drbd/etc/drbd.d/r0.res
写入：
resource r0 {                     //此处名字与文件铭r0.res对应
  on greencloud1{           //主机名
    device    /dev/drbd1;   //这是第一次出现，官方文档要求是以drbd开头，应该属于裸设备
    disk      /dev/sda6;           //这是新分出来的sda6区，未格式化
    address   192.168.103.122:7789;
    meta-disk internal;
  }
  on greencloud2{
    device    /dev/drbd1;
    disk      /dev/sda6;
    address   192.168.103.123:7789;
    meta-disk internal;
  }
}
```
### 3.启动drbd

#### 3.1.载入drbd模块

```
modprobe drbd                                    //载入 drbd 模块
lsmod|grep drbd                                  //确认 drbd 模块是否载入
```

#### 3.2.安装 cman 底层消息通讯 + 全局锁功能

```
ccs_tool create gfscluster
ccs_tool addnode -n 1 -v 1 greencloud1
ccs_tool addnode -n 2 -v 1 greencloud2
查看节点
ccs_tool lsnode
启动cman前：
service NetworkManager stop
chkconfig NetworkManager off
service cman restart
```

**注意：在启动cman的时候由于quorum无响应可能会导致cman启动失败。**

解决方法：
修改cman里面的quorum时间：

```
vi /etc/init.d/cman
[ -z "$CMAN_QUORUM_TIMEOUT" ] && CMAN_QUORUM_TIMEOUT=0
```

#### 3.3启动 drbd 服务

```
drbdadm create-md r0
drbdadm up r0
```

#### 3.4.两台机器都设置为主节点

```
drbdadm primary --force r0
```

#### 3.5.格式化gfs2并多处挂载

```
mkfs.gfs2 -j 2 -p lock_dlm -t gfscluster:gfslocktable /dev/drbd1
mount /dev/drbd1 /db
```
这时两台主机已经实现了双主模式了，在一台写入数据或者修改数据，另一台应该可以实时同步过去。

***
2014-09-25 17:31    bh xzl
