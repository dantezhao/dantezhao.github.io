---
layout: post
title:  "Linux普通用户执行root权限的程序"
categories: 不折腾被变羊
tags:  Linux
---

* content
{:toc}

## 前言

hadoop集群用了haproxy做负载均衡，但是我只有普通用户的权限，haproxy执行的时候又需要root权限，每次修改一下配置文件，还要找运维重启一下，忧桑的很，忽然想起来linux里面有suid和sgid的概念，记不清是细节，不过试了一下果真可行。

做个简单记录。




## suid和sgid

解决方法很简单，就是下面一句：

```
chmod u+s /user/sbin/haproxy
```

然后就会看到执行文件的权限变化了：

```
$ ll /usr/sbin/haproxy
-rwsr-xr-x 1 root root 734712 10月 16 2014 /usr/sbin/haproxy
```
**SUID：**

为了方便普通用户执行一些特权命令，SUID/SGID程序允许普通用户以root身份暂时执行该程序，并在执行结束后再恢复身份。

chmod u+s 就是给某个程序的所有者以suid权限，可以像root用户一样操作。

## 总结

linux水很深，想彻底玩明白不容易啊。

***
2016-06-28 hzct
