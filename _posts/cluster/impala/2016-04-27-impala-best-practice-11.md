---
layout: post
title:  "Impala实践之十一：parquet性能测试"
categories: 漫步云端
tags: Impala Parquet
---

* content
{:toc}

## 前言

之前一直考虑更换impala的文件存储格式为parquet，但是没有立即使用，最近又做了一些测试，看看parquet是否真的有用。在测试的时候顺便测了一下compute语句的效果，一起作为参考。下面抽出一个小业务的部分测试结果来展示。




## 测试准备

库名和表名当然不是真的。

### **测试范围：**
- 文件格式：parquet和text
- compute语句的影响

### **测试用表：**

| 表名        | 行数           | 字段数  | 物理存储大小 |
| ------------- |:-------------:| -----:| -----:|
| ain      | 34231137 | 11 | 1.4 G |
| a_in      | 395857172 | 11 | 4.4 G |
| in     | 62025197 | 6 | 2.5 G |
| c     | 4055068 | 144 | 708.3 M |


## 测试用例1

这个记录是当时随手测的一个结果。

### **sql语句：**

```
select count(*) from c;
```

### **测试结果：**

| 文件格式  | 第1次执行耗时   |第2次执行耗时    |
| ------------- |:-------------:|:-------------:|
| text      | 7.72s | 0.74s |
| parquet      |  5.90s | 0.53s |


## 测试用例2


### **sql语句：**

```
select count(uid) from c
where ***
```

### **测试结果：**

| 文件格式  | where字句数量   | 持续时间| 读取hdfs字节数 |累积内存使用峰值|
| ------------- |:-------------:|:-------------:| :-------------:|
| text      | 1 | 826ms |3G|361.1M|
| parquet      | 1 |623ms |17.1 M|6.9M|
| text      | 2 | 1.04s |3G|112.3 M|
| parquet      | 2 |623ms |17.6 M|7.1 M|
| text      | 3 | 930ms |3G|112.3 M|
| parquet      | 3 |631ms |18.4 M|7.6 M|
| text      | 6 | 961ms |3G|120.6 M|
| parquet      | 6 |836ms |22.9 M|45.1 M|
| text      | 13 | 1.04s |3G|117 M|
| parquet      | 13 |1.04s |33.2 M |19.5 M|

## 测试用例3

### **sql语句：**

dev表是另外一个表，不是parquet格式。

```
SELECT SUBSTR(a1.dt,1,7) dt, COUNT(DISTINCT a1.uid)
FROM (
SELECT userid uid , createtime dt
FROM dev) a1
LEFT JOIN (
SELECT uid, dt
FROM (
SELECT userid uid, time dt FROM a_in
UNION ALL
SELECT uid uid, stime dt FROM ain
WHERE atype='1'
UNION ALL
SELECT uid, time dt
FROM c
WHERE state!=0 AND source='test') a1 ) a2
ON a1.uid = a2.uid AND SUBSTR(a1.dt,1,7)>SUBSTR(a2.dt,1,7)
LEFT JOIN (
SELECT uid, dt
FROM (SELECT userid uid, time dt FROM in
UNION ALL
SELECT uid, time dt FROM c
WHERE state!=0 AND source='pc') a1 ) a3
ON a1.uid = a3.uid AND SUBSTR(a1.dt,1,7)>SUBSTR(a3.dt,1,7)
WHERE a2.uid IS NULL AND a3.uid IS NOT NULL
GROUP BY dt
ORDER BY dt;
```


### **测试结果：**

| 文件格式  | 持续时间 | 读取hdfs字节数 | 累积内存使用峰值 |
| ------------- |:-------------:| :-------------:| :-------------:|
| text      | 12分38秒 | 71.4G| 27.5G |
| parquet      |  12分27秒 | 22.5G | 27.6G |


## 测试用例4

这个稍微复杂一些，用到了上面的三张表，有一些join操作。因为前段时间发现了compute语句的神奇，因此这次顺便带上它。

### **sql语句：**

```
select SUBSTR(a1.dt,1,7) dt, COUNT(DISTINCT a1.uid)
FROM (
SELECT uid, createtime dt
FROM c
WHERE state!=0 AND source='app') a1
INNER JOIN (
SELECT uid, dt
FROM (
SELECT userid uid, logtime dt FROM a_in
UNION ALL
SELECT uid uid, stime dt FROM ain
WHERE atype='1') a1 ) a2
ON a1.uid = a2.uid AND SUBSTR(a1.dt,1,7) = SUBSTR(a2.dt,1,7)
GROUP BY dt
ORDER BY dt
```

### **测试结果：**



| 文件格式  | 提前执行compute  | 持续时间 | 读取hdfs字节数 | 累积内存使用峰值 |
|:-------------:|:-------------:|:-------------:| :-------------:| :-------------:|
| text      | N | 5分16秒 | 46.7G | 12.1G |
| parquet      | N | 3分48秒 | 1.7G| 27.3G |
| text      | Y | 34.9秒 | 46.7G | 1.5G |
| parquet      | Y | 14.5秒 | 1.7G| 1.1G |

***
2016-04-27 14:55:00 hzct
