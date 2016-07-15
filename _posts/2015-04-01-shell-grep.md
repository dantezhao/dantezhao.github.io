---
layout: post
title:  "Shell：grep/egrep/fgrep"
categories: Shell
tags:  linux shell grep 
---

* content
{:toc}

## grep

### 基本说明

grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

功能：搜索文本

语法：grep [选项] [模式] [文本]

grep也常和管道结合使用。

**个人理解，用好grep的关键在于用好正则表达式**





### 常用选项

- -? 同时显示匹配行上下的？行，如：grep -2 pattern filename同时显示匹配行的上下2行。
- -b，--byte-offset 打印匹配行前面打印该行所在的块号码。
- -c,--count 只打印匹配的行数，不显示匹配的内容。
- -f File，--file=File 从文件中提取模板。空文件中包含0个模板，所以什么都不匹配。
- -h，--no-filename 当搜索多个文件时，不显示匹配文件名前缀。
- -i，--ignore-case 忽略大小写差别。
- -q，--quiet 取消显示，只返回退出状态。0则表示找到了匹配的行。
- -n，--line-number 在匹配的行前面打印行号。
- -v，--revert-match 反检索，只显示不匹配的行。

### 例子

#### ll和grep
```

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
```

#### grep匹配文件内容

```
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
```



## egrep 和 fgrep

egrep和fgrep是grep加了额外的参数得到的命令，都算是在grep基础上行的一个封装。

man grep有一段解释：

>In  addition,  two  variant programs egrep and fgrep are available.  egrep is the same as grep -E.  fgrep is the same as grep -F.  Direct invocation as either egrep or fgrep is deprecated,  but  is  provided  to allow  historical  applications that rely on them to run unmodified.

### egrep

可以使用基本的正则表达外, 还可以用扩展表达式.

egrep = grep -E

> -E, --extended-regexp Interpret  PATTERN  as  an  extended  regular expression  (ERE, see below).    (-E is specified by POSIX.)

#### 扩展表达式

- `+` 匹配一个或者多个先前的字符, 至少一个先前字符.

- `?` 匹配0个或者多个先前字符.

- `a|b|c ` 匹配a或b或c

- `()` 字符组, 如: `love(able|ers)` 匹配loveable或lovers.

- `(..)(..)\1\2` 模板匹配. `\1`代表前面第一个模板, `\2`代第二个括弧里面的模板.

- `x{m,n} =x\{m,n\}` x的字符数量在m到n个之间.

### fgrep

速度比较快。

个人感觉可以理解为就是固化表达式的搜索,在不需要任何表达式和变量等的情况下，可以使用。

fgrep = grep -F

> -F, --fixed-strings Interpret PATTERN as a list of fixed strings, separated by newlines, any of which is to  be matched.  (-F is specified by POSIX.)

### grep/fgrep/egrep三者对比


下面是个简单的对比例子

```java

[root@z1 shell]# cat fgreptest 
[1-9].*
123142324
zhao
123abc456
^[1-9].*[6]$


//使用fgrep得到的是通过字符串匹配出来的结果
[root@z1 shell]# cat fgreptest | fgrep '^[1-9].*[6]$'
^[1-9].*[6]$

//使用grep和egrep得到是正则表达式的结果
[root@z1 shell]# cat fgreptest | egrep '^[1-9].*[6]$'
123abc456
[root@z1 shell]# cat fgreptest | grep '^[1-9].*[6]$'
123abc456

//egrep还可以通过扩展表达式得到以4或者6结尾的字符串，但是grep不行
[root@z1 shell]# cat fgreptest | grep '^[1-9].*(4|6)$'
[root@z1 shell]# cat fgreptest | egrep '^[1-9].*(4|6)$'
123142324
123abc456

```


******
2015-04-01 23:14:54 于bh xzl


