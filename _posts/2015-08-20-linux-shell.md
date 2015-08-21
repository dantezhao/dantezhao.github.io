---
layout: post
author: zhao
title:  Linux：Shell相关主要命令整理
date:   2015-08-20 23:14:54
categories: Linux
---

* content
{:toc}

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

~~~java
//-name 查找目录下名字为demo1的文件，包括子目录
[root@z1 shell]# find . -name demo1
./dir/demo1
./demo1

//-name 找到目录下符合正则表达式的文件
//正则表达式的内容需要在单引号内
[root@z1 shell]# find . -name '*mo*'
./dir/demo1
./demo1
./demo2

//找到权限是755的文件
//可以看到结果里面包含目录
[root@z1 shell]# find ./ -perm 755
./
./dir
./dir/demo1
./dir/demo1/demo3

//找到权限是755的文件，设定文件类型是f
[root@z1 shell]# find ./ -type f -perm 755
./dir/demo1/demo3

//查找属于用户组zhdd的文件
[root@z1 shell]# find . -group zhdd
./greptest

//查找用户root的文件
[root@z1 shell]# find . -user root
.
./.greptest.txt.swp
./dir
./dir/demo1
./dir/demo1/demo3
./demo1
./greptest
./demo2

//查找一天之内改过的文件
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

//查找5天前修改过的文件
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

//查找文件后m并执行cat命令
[root@z1 shell]# find ./ -name 'gre*' -exec cat {} \;
fslkajfjsalfkjsfjlsf
~~~















###grep

####基本说明

grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

功能：搜索文本

语法：grep [选项] [模式] [文本]

grep也常和管道结合使用。

**个人理解，用好grep的关键在于用好正则表达式**

####常用选项

-? 同时显示匹配行上下的？行，如：grep -2 pattern filename同时显示匹配行的上下2行。
-b，--byte-offset 打印匹配行前面打印该行所在的块号码。
-c,--count 只打印匹配的行数，不显示匹配的内容。
-f File，--file=File 从文件中提取模板。空文件中包含0个模板，所以什么都不匹配。
-h，--no-filename 当搜索多个文件时，不显示匹配文件名前缀。
-i，--ignore-case 忽略大小写差别。
-q，--quiet 取消显示，只返回退出状态。0则表示找到了匹配的行。
-n，--line-number 在匹配的行前面打印行号。
-v，--revert-match 反检索，只显示不匹配的行。

####例子

#####ll和grep
~~~java

//在ll显示的内容中查看包含log的行
//下面是两种方式
[root@z1 log]# ll |grep log
-rw-r--r--  1 root root   2199 Aug 17 11:48 boot.log
-rw-r--r--. 1 root root 167704 Aug 17 11:47 dracut.log
-rw-r--r--  1 root root    397 Aug 17 11:48 gshell.log
-rw-r--r--. 1 root root 146292 Aug 21 15:37 lastlog
-rw-------. 1 root root      0 Feb  4  2015 maillog
-rw-r--r--  1 root root    667 Aug 17 12:19 ntp.log
-rw-r--r--  1 root root    106 Aug 17 11:48 rdate.log
-rw-r--r--  1 root root    125 Aug 17 11:46 repair.log
-rw-------. 1 root root      0 Feb  4  2015 tallylog
-rw-r--r--  1 root root      0 Aug 17 11:48 vmstart.log
-rw-------  1 root root    816 Aug 21 10:09 xrdp-sesman.log
-rw-------. 1 root root  15251 Aug 20 19:11 yum.log

[root@z1 log]# ll |grep .*.log
-rw-r--r--  1 root root   2199 Aug 17 11:48 boot.log
-rw-r--r--. 1 root root 167704 Aug 17 11:47 dracut.log
-rw-r--r--  1 root root    397 Aug 17 11:48 gshell.log
-rw-r--r--. 1 root root 146292 Aug 21 15:37 lastlog
-rw-------. 1 root root      0 Feb  4  2015 maillog
-rw-r--r--  1 root root    667 Aug 17 12:19 ntp.log
-rw-r--r--  1 root root    106 Aug 17 11:48 rdate.log
-rw-r--r--  1 root root    125 Aug 17 11:46 repair.log
-rw-------. 1 root root      0 Feb  4  2015 tallylog
-rw-r--r--  1 root root      0 Aug 17 11:48 vmstart.log
-rw-------  1 root root    816 Aug 21 10:09 xrdp-sesman.log
-rw-------. 1 root root  15251 Aug 20 19:11 yum.log

//在ll显示的内容中查看包含log的行，用颜色区分，并添加行号
//shell中的结果是带颜色的
[root@z1 log]# ll |grep log --color
[root@z1 log]# ll |grep log --color -n
3:-rw-r--r--  1 root root   2199 Aug 17 11:48 boot.log
9:-rw-r--r--. 1 root root 167704 Aug 17 11:47 dracut.log
11:-rw-r--r--  1 root root    397 Aug 17 11:48 gshell.lo
12:-rw-r--r--. 1 root root 146292 Aug 21 15:37 lastlog
13:-rw-------. 1 root root      0 Feb  4  2015 maillog
15:-rw-r--r--  1 root root    667 Aug 17 12:19 ntp.log
19:-rw-r--r--  1 root root    106 Aug 17 11:48 rdate.log
20:-rw-r--r--  1 root root    125 Aug 17 11:46 repair.lo
25:-rw-------. 1 root root      0 Feb  4  2015 tallylog
26:-rw-r--r--  1 root root      0 Aug 17 11:48 vmstart.log
28:-rw-------  1 root root    816 Aug 21 10:09 xrdp-sesman.log
29:-rw-------. 1 root root  15251 Aug 20 19:11 yum.log


//统计匹配的行数
[root@z1 log]# ll |grep log -c
12
~~~

#####grep匹配文件内容

~~~java
//匹配文件中以数字开头以英语字母c结尾的行，并打印行号
[root@z1 log]# grep '^[0-9].*c$' ntp.log -n 
10:17 Aug 12:19:15 ntpd[895]: 0.0.0.0 c615 05 clock_sync


//匹配所有以.log结尾的文件满足正则表达式的行
[root@z1 log]# grep '^[A-Z].*11.*x86_64$' *.log
dracut.log:Mon Aug 17 11:47:07 CST 2015 Info: Executing /sbin/dracut -f --add-drivers xen-blkfront xen-blkfront virtio_blk virtio_blk virtio_pci virtio_pci virtio_console virtio_console initramfs-2.6.32-431.23.3.el6.x86_64.img 2.6.32-431.23.3.el6.x86_64
yum.log:Aug 20 19:06:13 Updated: openssh-5.3p1-112.el6_7.x86_64
yum.log:Aug 20 19:06:22 Installed: rp-pppoe-3.10-11.el6.x86_64
yum.log:Aug 20 19:06:29 Updated: libX11-1.6.0-6.el6.x86_64
yum.log:Aug 20 19:06:39 Installed: 1:dbus-x11-1.2.24-8.el6_6.x86_64
yum.log:Aug 20 19:06:43 Installed: xorg-x11-xkb-utils-7.7-4.el6.x86_64
yum.log:Aug 20 19:06:43 Installed: xorg-x11-server-common-1.15.0-36.el6.centos.x86_64
yum.log:Aug 20 19:06:44 Installed: 1:xorg-x11-xauth-1.0.2-7.1.el6.x86_64
yum.log:Aug 20 19:06:51 Installed: ConsoleKit-x11-0.4.1-3.el6.x86_64
yum.log:Aug 20 19:06:51 Installed: xorg-x11-server-utils-7.7-2.el6.x86_64
yum.log:Aug 20 19:06:52 Installed: xorg-x11-xinit-1.0.9-14.el6.x86_64
yum.log:Aug 20 19:06:57 Installed: mesa-dri1-drivers-7.11-8.el6.x86_64
yum.log:Aug 20 19:07:01 Installed: xorg-x11-drv-void-1.4.0-23.el6.x86_64
yum.log:Aug 20 19:07:01 Installed: xorg-x11-drv-vesa-2.3.2-15.el6.x86_64
yum.log:Aug 20 19:07:01 Installed: xorg-x11-server-Xorg-1.15.0-36.el6.centos.x86_64
yum.log:Aug 20 19:07:01 Installed: xorg-x11-drv-evdev-2.8.2-4.el6.x86_64
yum.log:Aug 20 19:07:01 Installed: xorg-x11-drv-wacom-0.23.0-4.el6.x86_64
yum.log:Aug 20 19:07:07 Installed: libgnome-2.28.0-11.el6.x86_64
yum.log:Aug 20 19:07:08 Installed: gnome-desktop-2.28.2-11.el6.centos.x86_64
yum.log:Aug 20 19:07:11 Installed: 1:control-center-2.28.1-39.el6.x86_64
yum.log:Aug 20 19:07:28 Installed: gnome-terminal-2.31.3-11.el6_6.x86_64
yum.log:Aug 20 19:07:32 Installed: pulseaudio-module-x11-0.9.21-21.el6.x86_64
yum.log:Aug 20 19:07:32 Installed: openssh-askpass-5.3p1-112.el6_7.x86_64
yum.log:Aug 20 19:07:33 Updated: openssh-server-5.3p1-112.el6_7.x86_64
yum.log:Aug 20 19:07:33 Updated: openssh-clients-5.3p1-112.el6_7.x86_64
yum.log:Aug 20 19:11:36 Installed: 1:xorg-x11-font-utils-7.2-11.el6.x86_64
yum.log:Aug 20 19:11:39 Installed: tigervnc-server-1.1.0-16.el6.centos.x86_64
yum.log:Aug 20 19:11:53 Installed: xrdp-0.6.1-4.el6.x86_64
~~~
