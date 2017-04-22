---
layout: post
title:  "Hadoop集群性能测试"
categories: 漫步云端
tags: Hadoop
---

* content
{:toc}


## 前言

### 测试方法

临时的一个小测试，主要目的是测试一下集群的IO。现从两方面进行测试：系统级别和集群级别。




### 集群环境

- 10台物理机，64G内存，2T硬盘。
- cdh5.x

### 工具

测试过程中使用到的工具。

- dd
- hdparm
- iperf
- hadoop benchmark

## 系统级别测试

通过对集群节点测试，块写40G耗时49秒，磁盘写IO 873MB/s，读IO 1022.49MB/s，点对点网络IO大概110MB/s

### 磁盘IO

磁盘写：

```
time dd if=/dev/zero of=/data/test.txt bs=1M count=40960
40960+0 records in
40960+0 records out
42949672960 bytes (43 GB) copied, 49.1891 s, 873 MB/s

real    0m49.201s
user    0m0.012s
sys    0m44.990s

```

磁盘读：

```
# hdparm -tT --direct /dev/vdb1

/dev/vdb1:
 Timing O_DIRECT cached reads:   3286 MB in  2.00 seconds = 1613.15 MB/sec
 Timing O_DIRECT disk reads: 3000MB in  3.01 seconds = 1022.49 MB/sec
```

### 网络IO

网络传输，点对点copy，传输速度平均101.6MB/s

iperf测的平均网络IO为110左右MB/s

## Hadoop Benchmark

### Benchmark工具

网上的benchmark工具挺多的，总结一下大致有下面几个：

- hadoop自带的Test
- intel的 HiBench
- 中科院的BigDataBench
- berkeley的benchmark
- ebay的benchmark（名字记不清了）

这是目前我找到的几个比较出名一些的hadoopbenchmark。缩小一下范围后，准备在前三个中选一个。其实这个各有特点，但是考虑到这次只测试io，而且集群的root权限也不在我这，就用个比较省事的，hadoop自带的了。


### 脚本

写了个小脚本。

```
jar_path=hadoop-test-mr1.jar
main_class=TestDFSIO

echo "开始hadoop集群测试！"
echo "-------------------------------------------------------------"

echo "清空测试目录！"

hadoop jar $jar_path $main_class -clean

echo "开始极小文件测试!"
echo "-------------------------------------------------------------"

echo "读写10000个10B的文件"
hadoop jar $jar_path $main_class -write -nrFiles 1000 -size "10B"
hadoop jar $jar_path $main_class -read -nrFiles 1000 -size "10B"

......

hadoop jar $jar_path $main_class -clean

echo "开始巨文件测试!"
echo "-------------------------------------------------------------"

echo "读写5个100G的文件"
hadoop jar $jar_path $main_class -write -nrFiles 5 -size "100GB"
hadoop jar $jar_path $main_class -read -nrFiles 5 -size "100GB"

```

### 测试结果

每一次测试都会在当前目录的`TestDFSIO_results.log`中追加新的测试结果。

```
----- TestDFSIO ----- : write
           Date & time: Tue Apr 12 12:20:18 CST 2016
       Number of files: 1000
Total MBytes processed: 0.009536743
     Throughput mb/sec: 9.813281434897923E-5
Average IO rate mb/sec: 9.844686428550631E-5
 IO rate std deviation: 5.294680350263851E-6
    Test exec time sec: 184.055

----- TestDFSIO ----- : read
           Date & time: Tue Apr 12 12:23:37 CST 2016
       Number of files: 1000
Total MBytes processed: 0.009536743
     Throughput mb/sec: 0.0029361893978024937
Average IO rate mb/sec: 0.003687877906486392
 IO rate std deviation: 0.002046490931134166
    Test exec time sec: 184.024
```

### 图

出的测试结果图就不上了。

***
2016-04-12 15:13:00 hzct
