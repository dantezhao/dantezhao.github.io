---
layout: post
title:  "Shell：遍历输出日期范围（指定两个日期间的所有日期）"
categories: Linux
tags:  linux shell
---

* content
{:toc}

## 前言

不是天天写脚本，以前会的脚本，现在都写不出来了，还是留个记录吧。




## 脚本

```
#!/usr/bin/env bash

date=`date -d "+0 day $1" +%Y-%m-%d`
enddate=`date -d "+1 day $2" +%Y-%m-%d`

echo "------------------------------"
echo "date=$date"
echo "enddate=$enddate"
echo "------------------------------"


while [[ $date < $enddate  ]]
do
    echo $date
    date=`date -d "+1 day $date" +%Y-%m-%d`
done
```

示例：

```
# sh date.sh 2016-05-25 2016-06-01
------------------------------
date=2016-05-25
enddate=2016-06-02
------------------------------
2016-05-25
2016-05-26
2016-05-27
2016-05-28
2016-05-29
2016-05-30
2016-05-31
2016-06-01

```



******
2016-07-07 11:11:11 hzct
