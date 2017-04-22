---
layout: post
title:  "Impala实践之七：添加负载均衡"
categories: 漫步云端
tags: Impala Haproxy
---

* content
{:toc}


## 前言

impala的负载均衡，使用haproxy来做，主要是比较简单。安装后做一个小配置就行。

主要用的就是haproxy四层交换机的特性，讲所有指向haproxy主机和端口的请求，转发到相应的主机：端口上。

cdh官网里面的信息已经比较久了，有些配置需要改，因此做一个笔记。




## impala负载

### 安装haproxy

```
yum install haproxy
```

### 配置文件

```
vim /etc/haproxy/haproxy.cfg
```

下面是一个实际的配置文件，主机信息已经隐藏。

```
global
    # To have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local0
    log         127.0.0.1 local1 notice
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    #stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#
# You might need to adjust timing values to prevent timeouts.
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    maxconn                 3000

    #连接时间需要修改，因为如果时间较短的话会出现，任务在执行，但是连接已经断开所以获取不到结果的情况。
    timeout connect 3600000ms
    timeout client 3600000ms
    timeout server 3600000ms

#
# This sets up the admin page for HA Proxy at port 25002.
#
listen stats :25002
    balance
    mode http
    stats enable
    stats auth admin:admin

# This is the setup for Impala. Impala client connect to load_balancer_host:25003.
# HAProxy will balance connections among the list of servers listed below.
# The list of Impalad is listening at port 21000 for beeswax (impala-shell) or original ODBC driver.
# For JDBC or ODBC version 2.x driver, use port 21050 instead of 21000.

# 注意：haproxy-host-ip是haproxy的ip地址，主机名也可以
listen impala haproxy-host-ip:25003

    # impala负载均衡需要第四层的，所以填tcp
    mode tcp
    option tcplog
    balance leastconn

    #主机列表
    #impala-host-1是impala的主机列表
    #由于是要对jdbc的请求进行转发，所以端口设置的是21050
    server impala-1 impala-host-1:21050
    server impala-2 impala-host-2:21050
    server impala-3 impala-host-3:21050
    server impala-4 impala-host-4:21050
```

### 启动

```
/usr/sbin/haproxy  -f   /etc/haproxy/haproxy.cfg
```

### 使用

使用起来十分方便，区别仅仅相当于是修改了一个ip地址和端口而已，其余不变。

```
jdbc:hive2://haproxy-host-ip:25003/;auth=noSasl
```

## 总结

网络知识基本上是废的差不多了，之前四层交换机都想不起来是什么东东。重新稍微温习一下基础，在脑子里面就会对这些负载均衡还是什么几层交换机理解了，再来做工程，至少是从原理上能有所把握，不至于是为了做东西而做东西，一堆堆的组件拼接。


***
2016-04-21 15:17:00 hzct
