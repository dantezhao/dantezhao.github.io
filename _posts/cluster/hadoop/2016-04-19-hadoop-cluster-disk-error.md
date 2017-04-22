---
layout: post
title:  "一次集群硬盘故障记录"
categories: 漫步云端
tags: Hadoop
---

* content
{:toc}


## 前言

早上发集群中一台物理机的磁盘故障。发现问题后先做了一些操作，比如停掉这台机器的服务。




## 解决过程

**先看一下集群的状态**

```
hadoop dfsadmin -report
```

```

Configured Capacity:
Present Capacity:
DFS Remaining:
DFS Used:
DFS Used%: 51.48%
Under replicated blocks:
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (7):

Name: ip:50010 (hostname)
Hostname: hostname
Rack: /default
Decommission Status : Normal
Configured Capacity:
DFS Used:
Non DFS Used:
DFS Remaining:
DFS Used%: 47.22%
DFS Remaining%: 47.20%
Configured Cache Capacity: 4294967296 (4 GB)
Cache Used: 0 (0 B)
Cache Remaining: 4294967296 (4 GB)
Cache Used%: 0.00%
Cache Remaining%: 100.00%
Xceivers: 31
Last contact: Tue Apr 19 10:58:54 CST 2016

......

Dead datanodes (1):

Name: ip:50010 (dead_hostname)
Hostname: dead_hostname
Rack: /default
Decommission Status : Decommission in progress
Configured Capacity: 0 (0 B)
DFS Used: 0 (0 B)
Non DFS Used: 0 (0 B)
DFS Remaining: 0 (0 B)
DFS Used%: 100.00%
DFS Remaining%: 0.00%
Configured Cache Capacity: 4294967296 (4 GB)
Cache Used: 0 (0 B)
Cache Remaining: 4294967296 (4 GB)
Cache Used%: 0.00%
Cache Remaining%: 100.00%
Xceivers: 0
Last contact: Tue Apr 19 10:13:27 CST 2016
```


**执行`hadoop fsck /`**

会发现报错很多缺失块的问题，因为还抱有幻想希望盘能修好，因此没有执行`hadoop fsck --delete`操作。

```
/user/root/test.txt:  Under replicated BP-1069288-141-1454466502803:blk_1074786045275. Target Replicas is 2 but found 1 replica(s).
/user/root/test1.txt:  Under replicated BP-10619288-10.54466502803:blk_1091327_2852203. Target Replicas is 3 but found 2 replica(s).
..
......

Status: HEALTHY
 Total size:	6478807583062 B (Total open files size: 12661477 B)
 Total dirs:	74799
 Total files:	1028230
 Total symlinks:		0 (Files currently being written: 169)
 Total blocks (validated):	1045666 (avg. block size 6195867 B) (Total open file blocks (not validated): 166)
 Minimally replicated blocks:	1045666 (100.0 %)
 Over-replicated blocks:	0 (0.0 %)
 Under-replicated blocks:	209121 (19.998833 %)
 Mis-replicated blocks:		0 (0.0 %)
 Default replication factor:	2
 Average block replication:	1.8725061
 Corrupt blocks:		0
 Missing replicas:		209121 (9.649644 %)
 Number of data-nodes:		7
 Number of racks:		1
FSCK ended at Tue Apr 19 11:19:12 CST 2016 in 50218 milliseconds


The filesystem under path '/' is HEALTHY
```

过了段时间后，运维把盘搞定了，听说是什么snapshot的问题。然后重新加节点加入集群，这个时候再次执行`hadoop fsck /`操作，会发现，每次执行的时候namenode都能发现更多的数据块，也就是说丢失的数据都找回来了。最后的一次执行结果就是没有块丢失了。

```
....................................................................................................
....................................................................................................
....................................................................................................
....................................................................................................
....................................................................................................
....................................................................................................
.....Status: HEALTHY
 Total size:
 Total dirs:
 Total files:
 Total symlinks:		0 (Files currently being written: )
 Total blocks (validated):
 Minimally replicated blocks:	(99.99999 %)
 Over-replicated blocks:	0 (0.0 %)
 Under-replicated blocks:	0 (0.0 %)
 Mis-replicated blocks:		0 (0.0 %)
 Default replication factor:	2
 Average block replication:	2.072726
 Corrupt blocks:		0
 Missing replicas:		0 (0.0 %)
 Number of data-nodes:		8
 Number of racks:		1
FSCK ended at Tue Apr 19 11:41:58 CST 2016 in 13683 milliseconds


The filesystem under path '/' is HEALTHY
```

## 总结

集群使用的是2备份，这样能极大的减少集群的使用空间，但是也有着数据丢失的风险，因此下一步的一个策略是使用一台大容量的服务器做冷备份，每天定时将重要数据备份到该机器上。


***
2016-04-19 12:17:00 hzct
