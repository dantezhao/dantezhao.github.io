---
layout: post
title:  "Hive：sql练习题之一（用户商品推荐）"
categories: 漫步云端
tags: Hive Sql
---

* content
{:toc}


## 前言

一道sql练习题，在hive上跑的，用到了hive的一些窗函数。




## 练习题

### 数据

建表语句：

```
drop table if exists `dante_test.rank`;
create external table `dante_test.rank` (
  `uid` int COMMENT '用户id',
  `product_skn` int COMMENT '商品id',
  `brand_id` int COMMENT '品牌id',
  `rank` int '排序'
)
row format delimited fields terminated by ','
location '/tmp/dante/rank';

load data inpath '/tmp/dante/rank' into table dante_test.rank
```

数据集

```
1,50001,11,1
1,50002,11,2
1,50003,12,3
1,50004,13,4
2,50002,11,1
2,50004,13,2
2,50005,13,3
2,50006,14,4
```

### 描述

就是对一个用户，在他偏好的商品中，把每个品牌中排第一的排前面，内部顺序还是依照原来的顺序；后面再跟着每个品牌内排第二的；再跟每个品牌内排第三的。

### 答案

```
select uid, product_skn, brand_id, dense_rank() over (distribute by uid sort by product_rank asc, rank asc) as rank from
 (select uid, product_skn, brand_id, rank, row_number() over (distribute by uid, brand_id sort by rank asc) as product_rank
    from dante_test.rank) as a
  order by uid asc, rank asc;
```

### 结果

```
Total MapReduce CPU Time Spent: 8 seconds 540 msec
OK
1	50001	11	1
1	50003	12	2
1	50004	13	3
1	50002	11	4
2	50002	11	1
2	50004	13	2
2	50006	14	3
2	50005	13	4

```

## 总结

比较惭愧，从导数据到最后把sql跑出来，花了一个多小时。

光看不练会感觉sql没什么，但是写的时候，这些窗函数都扔在脸上自己也写不出来，不折腾很难搞定。

不过最后踩点坑，总结点经验还是很开心的。


***
2016-05-19 20:15:00 hzct
