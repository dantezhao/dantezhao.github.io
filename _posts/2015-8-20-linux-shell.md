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

####详细说明

#####-exec

find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格

举例：找出当前目录下大小为0的文件，并执行ls指令

~~~
find ./ -size 0 -exec ls {} \;
~~~

#####**-name** 

按照文件名查找文件,支持正则表达式

~~~
find ./ -name demo1;
~~~


#####-perm

按照文件权限来查找文件，在当前目录下查找文件权限位为755的文件

~~~
find . -perm 755 -print
~~~

#####-user

按照文件属主来查找文件，在/home目录中查找文件属主为chinahadoop的文件

~~~
find /home -user chinahadoop -print
~~~

#####-group 

按照文件所属的组来查找文件，在/home目录下查找属于hadoop用户组的文件

~~~
find /home -group hadoop -print
~~~

#####-mtime -n +n 

按照文件的更改时间来查找文件， -n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。在系统当前目录下查找更改时间在5日以内的文件

~~~
find . -mtime -5 -print
~~~