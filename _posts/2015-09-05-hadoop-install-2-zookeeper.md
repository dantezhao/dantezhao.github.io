---
layout: post
author: zhao
title:  "Hadoop：集群安装之二（Zookeeper安装）"
date:   2015-09-05 10:14:54
categories: Hadoop
---

* content
{:toc}

##安装

###配置文件

zookeeper机器的数量应该是奇数台，我这里用了三台。

zookeeper只有两个配置文件需要修改：zoo.cfg和zkEnv.sh。

####zoo.cfg文件

~~~
dataLogDir=/mnt/data/zookeeper/logs  //存放log日志的目录
dataDir=/mnt/data/zookeeper/data	//存放zk信息的目录，其中myid就在其中
server.1=z1:2888:3888		//填写zk集群的信息
server.2=z2:2888:3888
server.3=z3:2888:3888
~~~

####zkEnv.sh

添加下面一行。

配置这一项的主要原因是因为，在启动zk的时候zookeeper.out文件会出现在你启动时所在的目录中，不方便管理，因此需要手动配置一下。

~~~
ZOO_LOG_DIR="/mnt/zhao/data/zookeeper/logs"
~~~

###创建myid文件

这个数字1，是和zoo.cfg配置文件中的`server.1=z1:2888:3888`中对应的。

~~~
echo 1 > /mnt/zhao/data/zookeeper/data/myid
~~~

###分发文件

我使用了clustershell来分发文件，直接scp单台机器发也是可以的。

~~~
clush -b -g zookeeper --copy /mnt/zhao/soft/zookeeper-3.4.6/ --dest /mnt/zhao/soft/
~~~

##启动

所有安装zookeeper的机器上执行

~~~
zkServer.sh start
~~~

然后验证

~~~
[hadoop@z1 zookeeper-3.4.6]$ zkServer.sh status
JMX enabled by default
Using config: /mnt/zhao/soft/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
~~~


