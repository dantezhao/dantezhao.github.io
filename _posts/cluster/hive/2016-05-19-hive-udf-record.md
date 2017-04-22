---
layout: post
title:  "Hive：窗函数和UDF整理"
categories: 漫步云端
tags: Hive
---

* content
{:toc}

## 前言

写sql的时候偶尔会遇到一些函数，做个小记录，备忘。




## UDFs

### **Conditional Functions**



| Return Type | Name(Signature)  | Description | 描述 | example |
|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
| T	| COALESCE(T v1, T v2, ...)	| Returns the first v that is not NULL, or NULL if all v's are NULL. | 可以用在控制uid为null的情况 | coalesce(a.uid, 0) as uid |
| T | CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END |When a = b, returns c; when a = d, returns e; else returns f.|
| T | CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END | When a = true, returns b; when c = true, returns d; else returns e.| 把计算结果赋值给end，并重命名为date_time | case when a = -1 then 0 else 1 end as date_time|

## 窗函数

## **Analytics functions**

| Return Type | Name(Signature)  | Description | 描述 | example |
|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
| T	| RANK()	|  | 生成数据项在分组中的排名，排名相等会在名次中留下空位 | RANK() OVER(PARTITION BY id ORDER BY name desc) AS rn1 |
| T	| DENSE_RANK() |  | 生成数据项在分组中的排名，排名相等会在名次中不会留下空位 |  |
| T	| ROW_NUMBER() |  | 从1开始，按照顺序，生成分组内记录的序列 | 比如，按照pv降序排列，生成分组内每天的pv名次:ROW_NUMBER() OVER(PARTITION BY id ORDER BY pv desc) AS rn |

## 参考

[Hive LanguageManual UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)
[Hive LanguageManual WindowingAndAnalytics](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+WindowingAndAnalytics)
[Hive分析函数系列文章](http://lxw1234.com/archives/2015/07/367.htm)
