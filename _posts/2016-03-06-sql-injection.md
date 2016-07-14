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

