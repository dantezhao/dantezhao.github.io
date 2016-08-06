---
layout: post
title:  "Impala实践之四：记一次Impala报错的处理和分析过程"
categories: 数据平台
tags: impala
---

* content
{:toc}

## 前言

impala集群出错的一次记录和解决方法以及解决思路。




## 错误记录

### 错误信息

```
Memory limit exceeded Cannot perform hash aggregation. Partitioned input data too many times. This could mean there is too much skew in the data or the memory limit is set too low.
```

### Query信息

就是个这么长的Query语句，Query需要join十多张的表，各种的字段。这只是很多sql中的其中一个。

```
create TABLE test.cp_ag_info ASSELECT a1.id cid, hr_num, position_num, available_po_num, rs_num, auto_filter_num, read_num, see_num, manual_refuse_num, it_num, auto_refuse_num, forward_num, get_rs_po_num, get_read_rs_po_num, get_see_rs_po_num, get_it_rs_po_num
FROM mysql.cp a1
LEFT JOIN (
SELECT cid, COUNT(DISTINCT uid) hr_num
FROM (
SELECT id uid, testid cid
FROM mysql.dante_user

......

UNION
SELECT a1.user_id uid, a2.dante_cp_id cid
FROM mds.t_cp_user a1
LEFT JOIN mds.t_cp a2
ON a1.cp_id=a2.id
WHERE a1.is_del='false' AND a2.is_del='false') f
GROUP BY cid) a6
ON CAST(a1.id AS STRING)= a6.cid
LEFT JOIN (
SELECT testid cid, COUNT(1) position_num, COUNT(CASE WHEN isenable!=0 AND isexpired!=1

......

COUNT(CASE WHEN a1.DELIVER_AUTO_FILTER=1 THEN a1.orderid END) auto_filter_num,
COUNT(CASE WHEN a1.READ_rs=1 THEN a1.orderid END) read_num,
COUNT(CASE WHEN a1.READ_CONTACT=1 THEN a1.orderid END) see_num,
COUNT(CASE WHEN a1.MANUAL_REFUSE=1 THEN a1.orderid END) manual_refuse_num,
COUNT(CASE WHEN a1.ONLINE_it=1 OR a1.OFFLINE_it=1 THEN a1.orderid END) it_num,
COUNT(CASE WHEN a1.AUTO_REFUSE=1 THEN orderid END) auto_refuse_num,
COUNT(CASE WHEN a1.AUTO_FORWARD=1 OR a1.MANUAL_FORWARD=1 THEN orderid END) forward_num
FROM test.ur a1
GROUP BY a1.testid) a8
ON a1.id=a8.cid
LEFT JOIN (
SELECT a1.testid cid,

......

a1.READ_rs=1 THEN a1.positionid END) get_read_rs_po_num
FROM test.ur a1
GROUP BY testid) a10
ON a1.id=a10.cid
LEFT JOIN (
SELECT a1.testid cid,
COUNT(DISTINCT CASE WHEN a1.READ_CONTACT=1 THEN a1.positionid END) get_see_rs_po_num
FROM test.ur a1
GROUP BY testid) a11
ON a1.id=a11.cid
LEFT JOIN (
SELECT a1.testid cid,
COUNT(DISTINCT CASE WHEN a1.ONLINE_it=1 OR a1.OFFLINE_it=1 THEN a1.positionid END)
......

ON a1.id=a12.cid
```

## 错误现象和解决方法

出现这个错误的原因非常奇葩，根据猜测是因为今天在给进群添加资源管理Llama时出现的，开启Llama然后关闭，它会修改impalad的资源上限，之前是32G的，结果被修改成了8G，而我还不知道被改了，也是看了很久才发现的。

今天在线上测试Llama后，因为感觉不太合适就关掉了，然后就开始出现各种的Memory Limit的错误，之前的正常运行的大Query今天集群失败，以前是没有错误的。定位后，修改一下大小就行了。

这个问题出现后，还出现过一次其它的问题，但是只出现了一次，不明白是什么原因，因为没有复现，所以没再处理。

```
Memory limit exceeded The memory limit is set too low initialize the spilling operator. The minimum required memory to spill this operator is 528.00 MB.
```

******
2016-04-05 19:53:00
