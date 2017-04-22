---
layout: post
title:  "Impala实践之八：脚本中引号问题"
categories: 漫步云端
tags: Impala Shell
---

* content
{:toc}

## 前言

写脚本，遇到一个小坑，python和seven帮忙填了一下，突然想起来之前貌似遇到过类似的情况。




## 版本一

脚本：

```
sql=$1
coordinator=$2
output_file=$3

echo $sql
echo "------"
echo $output_file
echo "------"
echo $coordinator

impala-shell -i $coordinator -q $sql -o $output_file
```

执行命令：
```
bash impala-exec.sh "select distinct dt from table" ip:21000 /tmp/test.txt
```

结果：
```
$ bash impala-exec.sh "select distinct dt from tablename" "ip1:21000" "tmp/test.txt"

select distinct dt from tablename
------
tmp/test.txt
------
ip1:21000

Error, could not parse arguments "distinct dt from tablename"
Usage: impala_shell.py [options]

Options:
  -h, --help            show this help message and exit
  -i IMPALAD, --impalad=IMPALAD
                        <host:port> of impalad to connect to
                        [default: hadoop-cluster-8-228:21000]
                ......
```

可以看到，脚本获取到了，但是impala-shell识别不了。

## 版本二

仔细想了一下，应该是impala在解析sql语句时候的语法规范问题，修改一下脚本。

改成如下即可。

```
impala-shell -i $coordinator -q "$sql" -o $output_file
```

***
2016-04-11 19:08:00 hzct
