---
layout: post
author: zhao
title:  Linux：Shell相关主要指令
date:   2015-08-20 23:14:54
categories: Linux
---


##命令总结

###find

####基本说明

功能：查找文件或目录

语法：find [选项] [目录或文件名称]

选项：
- options，指定find命令的常用选项
- print，find命令将匹配的文件输出到标准输出
- exec，find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格 

####常用选项

- -name 按照文件名查找文件

- -perm 按照文件权限来查找文件

- -user 按照文件属主来查找文件

- -group 按照文件所属的组来查找文件

- -mtime -n +n 按照文件的更改时间来查找文件， **-n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前**。

- -type 按照文件类型查找文件

	- b - 块设备文件。
	- d - 目录。
	- c - 字符设备文件。
	- p - 管道文件。
	- l - 符号链接文件。
	- f - 普通文件。
	
####例子

~~~html
#-name 查找目录下名字为demo1的文件，包括子目录
[root@z1 shell]# find . -name demo1
./dir/demo1
./demo1

#-name 找到目录下符合正则表达式的文件
#正则表达式的内容需要在单引号内
[root@z1 shell]# find . -name '*mo*'
./dir/demo1
./demo1
./demo2

#找到权限是755的文件
#可以看到结果里面包含目录
[root@z1 shell]# find ./ -perm 755
./
./dir
./dir/demo1
./dir/demo1/demo3

#找到权限是755的文件，设定文件类型是f
[root@z1 shell]# find ./ -type f -perm 755
./dir/demo1/demo3

#查找属于用户组zhdd的文件
[root@z1 shell]# find . -group zhdd
./greptest

#查找用户root的文件
[root@z1 shell]# find . -user root
.
./.greptest.txt.swp
./dir
./dir/demo1
./dir/demo1/demo3
./demo1
./greptest
./demo2

#查找一天之内改过的文件
[root@z1 log]# ll secure
-rw-------. 1 root root 3541 Aug 21 16:06 secure
[root@z1 log]# find -mtime -1
./cron
./wtmp
./sa
./sa/sa20
./sa/sar20
./sa/sa21
./audit/audit.log
./lastlog
./secure
[root@x129 log]# ll secure
-rw-------. 1 root root 3541 Aug 21 16:06 secure

#查找5天前修改过的文件
[root@x129 log]# find -mtime +5
./prelink

.....

./gdm/:0.log.4
./gdm/:0-slave.log.1
./gdm/:0-greeter.log.3
./gdm/:0-slave.log
./gdm/:0-greeter.log.2
./gdm/:0-slave.log.2
./gdm/:0-greeter.log
./gdm/:0.log.1
./gdm/:0-greeter.log.4
./gdm/:0-slave.log.3
./gdm/:0.log.2
./gdm/:0-greeter.log.1
[root@x129 log]# ll ./gdm/:0-greeter.log.1
-rw-r--r--. 1 gdm gdm 1218 Sep 25  2014 ./gdm/:0-greeter.log.1

#查找文件后m并执行cat命令
[root@z1 shell]# find ./ -name 'gre*' -exec cat {} \;
fslkajfjsalfkjsfjlsf
~~~













