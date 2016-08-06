---
layout: post
title:  "Impala实践之10：impala压缩方式测试"
categories: 数据平台
tags: impala compress
---

* content
{:toc}

## 前言

测一下parquet、snappy、gzip、textfile这些方式在hdfs中占用的存储大小。




在impala中直接建内部表。

## 测试

| 存储格式        | 压缩格式       |文件大小       |建表时间 |
|:-------------:|:-------------:|:-------------:|
| textfile     | none | 3.0 G | 38.74s |
| parquet      | none | 1.5 G |32.33s |
| parquet      | snappy | 709.3 M  | 31.71s |
| parquet     | gzip | 471.5 M | 48.23s |

## snappy

snappy的官方描述。

>Snappy is a compression/decompression library. It does not aim for maximum compression, or compatibility with any other compression library; instead, it aims for very high speeds and reasonable compression. For instance, compared to the fastest mode of zlib, Snappy is an order of magnitude faster for most inputs, but the resulting compressed files are anywhere from 20% to 100% bigger. On a single core of a Core i7 processor in 64-bit mode, Snappy compresses at about 250 MB/sec or more and decompresses at about 500 MB/sec or more.

## 补充

impala切换不同的压缩需要使用如下命令，在执行建表命令需要前用这个命令指定压缩方式。

```
set COMPRESSION_CODEC=gzip；
```

## 总结

impala在创建parquet表的时候已经默认了压缩格式为snappy，因此除非要修改为gzip或者不需要压缩，不用在进行其他的设置。

***
2016-05-04 19:55:30 hzct
