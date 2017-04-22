---
layout: post
title:  "Impala实践之十三：Impala建表时的关键字"
categories: 漫步云端
tags: Impala
---

* content
{:toc}

## 前言

由于经常要帮数据分析抽表，因此自己写了个自动生成impala和sqoop脚本的工具，结果今天发现一个库中17张表，只成功导入了12张。仔细检查才发现是是由于impala建表时候字段使用了location关键字的原因。

## 分析

建表语句
```
impala-shell -i ip:25004 -q "
DROP TABLE IF EXISTS database.table;
CREATE EXTERNAL TABLE database.table(
id string,
location string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS PARQUET
LOCATION '/path';
"
```

错误
```
ERROR: AnalysisException: Syntax error in line 3:
location string
^
Encountered: LOCATION
Expected: IDENTIFIER

CAUSED BY: Exception: Syntax error
Could not execute command: create EXTERNAL TABLE dante_test.testtable1(
id string,
location string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS PARQUET
LOCATION '/path'
```


经过几次试验，发现不管加什么符号，都会报错。

暂时的猜测是location占用了impala的关键字，只要遇到location就会认为是读到了设置hdfs路径的位置，因此就会出现语法错误。由于需求要的比较急，没有细究，暂时给表中的字段改了个名字。

***
2016-06-27 22:32:00 rljp
