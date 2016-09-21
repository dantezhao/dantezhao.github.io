---
layout: post
title:  "Hadoop清空回收站"
categories: 数据平台
tags: hdfs
---

* content
{:toc}


## 直接删除目录（不放入回收站）

```
hdfs dfs -rm -skipTrash /path/to/file/you/want/to/remove/permanently
```

如果不加`-skipTrash`，删除的目录会放入`/user/hdfs/.Trash`中。有专门的配置项来指定什么时候清空回收站。




## 清空回收站

```
hdfs dfs -expunge
```

> This should give you output similar to this. Basically in a minute the trash will be emptied.

执行完命令后，回收站的数据不会立即被清理，而是先打了一个checkpoint。显示的是一分钟后清除。

*实际验证，11T的数据需要好几分钟.....*

```
16/09/18 10:57:04 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 1 minutes, Emptier interval = 0 minutes.
16/09/18 10:57:04 INFO fs.TrashPolicyDefault: Created trash checkpoint: /user/hdfs/.Trash/160918105704
```

## 参考

http://centoshowtos.org/hadoop/emptying-the-hdfs-trash/

***
2016-09-18 12:39:00 hzct

***

转载请注明： 转载自 [**赵德栋的博客**](http://zhaodedong.com)

作者：赵德栋，[作者介绍](http://zhaodedong.com/about/)

本博客的文章集合：http://zhaodedong.com/category/
