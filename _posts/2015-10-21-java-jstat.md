---
layout: post
author: zhao
title:  Java工具：jstat
date:   2015-10-21 22:14:54
categories: Java
---

* content
{:toc}

##简介

###功能

一个极强的监视VM内存工具，可以用来监视VM内存内的各种堆和非堆的大小及其内存使用量。

jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”。

它位于java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。

###使用方法

jstat比较常用的使用方法：`jstat -gcutil ${PID} 2000`，这条指令的意思是，每隔2000的时间间隔统计GC的信息。

相信的使用方法如下：

~~~
[root@z1 ~]# jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as 
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.

~~~

jstat有很多可选的option，使用方法同上。

~~~
-class
-compiler
-gc
-gccapacity
-gccause
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcpermcapacity
-gcutil
-printcompilation
~~~

###参数介绍

常用的参数含义如下

- S0：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
- S1：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
- E：年轻代中Eden（伊甸园）已使用的占当前容量百分比
- O：old代已使用的占当前容量百分比
- P：perm代已使用的占当前容量百分比
- YGC：从应用程序启动到采样时年轻代中gc次数
- YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
- FGC：从应用程序启动到采样时old代(全gc)gc次数
- FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
- GCT：从应用程序启动到采样时gc用的总时间(s)

##例子

~~~
[root@z1 ~]# jstat -gcutil 872 2000
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
 22.60   0.00  95.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  95.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  95.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  97.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  97.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  97.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  97.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  99.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  99.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  99.94   0.00  11.42      2    0.003     0    0.000    0.003
 22.60   0.00  99.94   0.00  11.42      2    0.003     0    0.000    0.003
  0.00  21.78   2.02   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   2.02   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   2.02   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   2.02   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   4.04   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   4.04   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   4.04   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   4.04   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   6.05   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   6.05   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   6.05   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   6.05   0.00  11.42      3    0.005     0    0.000    0.005
  0.00  21.78   8.06   0.00  11.42      3    0.005     0    0.000    0.005
~~~







