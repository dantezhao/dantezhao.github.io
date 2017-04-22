---
layout: post
title:  "一次集群文件无故被删问题排查"
categories: 漫步云端
tags: Hadoop
---

* content
{:toc}

## 前言

发现集群部分目录文件丢失，但是都有一定的规律，比如该目录下，只留了七天的数据。这个事情挺严重的，需要排查一下具体问题。




```
[cluster]$ hadoop fs -ls /data/logs/log1
Found 6 items
drwxr-xr-x   - user supergroup          0 2016-07-15 01:13 /data/logs/log12016-07-14
drwxr-xr-x   - user supergroup          0 2016-07-16 01:12 /data/logs/log12016-07-15
drwxr-xr-x   - user supergroup          0 2016-07-17 01:11 /data/logs/log12016-07-16
drwxr-xr-x   - user supergroup          0 2016-07-18 01:11 /data/logs/log12016-07-17
drwxr-xr-x   - user supergroup          0 2016-07-19 01:13 /data/logs/log12016-07-18
drwxr-xr-x   - user supergroup          0 2016-07-20 01:12 /data/logs/log12016-07-19
```

## 排查思路和过程

我的思路主要是这样：先定位到哪台机器执行了这些操作，然后再在这台机器上找到登录的人和具体执行的脚本。

### 1. history 查看

通过history查看，在我们所有的机器上查看了一番，发现没有相关的命令，猜测可能是脚本，但是脚本的话，就更难找了。

### 2. crontab 查看

然后再查看crontab的情况。

各个用户看了一下，没发现什么明显带有删除命令的脚本

### 3.fsimage文件

上面两步都没用，就只有通过hadoop平台自身的命令了。

首先想到的就是fsimage文件，但是这里面只包含了文件的信息，没有操作记录。

```
hdfs oiv -i fsimage_#### -p XML -o f1.xml
```

```
<inode><id>16777220</id><type>FILE</type><name>data.146702594sad3850</name><replication>2</replication><mtime>1467025974881</mtime><atime>14670fds25943899</atime><perferredBlockSize>134217728</perferredBlockSi
ze><permission>user:supergroup:rw-r--r--</permission><blocks><block><id>1089131808</id><genstamp>15409803</genstamp><numBytes>672</numBytes></block>
</blocks>
</inode>
```

### 4.edits_#### 文件

然后就找到edits文件，但是edits文件里面只记录了一定时间内的操作记录，过了这个某个时间点就会被合并到fsimage中。最新的edits文件，没有找到相关目录的操作。

```
hdfs oev -i edits_#### -o e1.xml
```

```
<RECORD>
    <OPCODE>OP_ADD</OPCODE>
    <DATA>
      <TXID>1259598798</TXID>
      <LENGTH>0</LENGTH>
      <INODEID>22356685</INODEID>
      <PATH>/dir/dir/dir/dir/2016-07-20/dir/dir/file</PATH>
      <REPLICATION>2</REPLICATION>
      <MTIME>1468975996348</MTIME>
      <ATIME>1468975996348</ATIME>
      <BLOCKSIZE>134217728</BLOCKSIZE>
      <CLIENT_NAME>DFSClient_NONMAPREDUCE_1299808995_33</CLIENT_NAME>
      <CLIENT_MACHINE>ip地址</CLIENT_MACHINE>
      <OVERWRITE>true</OVERWRITE>
      <PERMISSION_STATUS>
        <USERNAME>plat</USERNAME>
        <GROUPNAME>supergroup</GROUPNAME>
        <MODE>420</MODE>
      </PERMISSION_STATUS>
      <RPC_CLIENTID>0931bf92-7d76-4e48-8914-73d6d5dbf4b0</RPC_CLIENTID>
      <RPC_CALLID>2793790</RPC_CALLID>
    </DATA>
  </RECORD>
```

### 5.偶然存在的一个脚本

这时候突然想到，我之前为了记录每天的数据新增量，写了个脚本，每天记录各个目录的数据大小。我通过这个脚本的变化，就能定位到该目录是哪天突然减少的，然后再查当天的日志就可以了。

幸运的是，的确找到了。

```
执行hadoop fs -du  -h /lagou
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
记一次hdfs容量。
当前时间是： 2016-07-19 09:00:00^M
执行hadoop fs -df -h /
Filesystem                           Size    Used  Available  Use%
hdfs://host:8020  xx T  xx T      xx T    xx %
执行hadoop fs -du  -h /
178.2 G  356.5 G  /dir1
0        0        /dir2
......
315.4 G  631.8 G  /dir4
28.3 G   58.8 G   /dir5
3.9 G    7.9 G    /dir6
```


### 6.每小时定时备份元数据

找到之后是哪一天的问题后，就是怎么获取那一天的edits文件了，这时候就是看你之前的人品了，我前段时间刚开始备份namenode的元数据信息，每隔1小时将所有的元数据信息备份一下，这次用上了~

通过一天24小时所有的文件操作日志，就能找到具体的操作ip了。

既然已经定位到了某个ip，那么就可以在该机器仔细找一下原因。

然后就找到了操作的时间。

### 7.定位执行脚本

再次仔细检查该机器上所有用户的crontab的脚本，以及调度任务里面的脚本，就找到了如下内容：

```
[]$ crontab -l
10 1 * * * /bin/sh /dir/daily.sh >> /dir/daily.log
```

```
if [ $opt == 'compress' ]; then

        echo "------------------"
        echo "压缩文件 /data/logs "
        echo "------------------"

        hadoop jar /dir/jar.jar 'compress' "$cnt_date" /data/logs

if [ $opt == 'delete' ]; then

        echo "------------------"
        echo "删除文件 /data/logs"
        echo "------------------"

        hadoop jar /dir/jar.jar 'delete' "$cnt_date" /data/logs
```

暂时定位到了该脚本，里面删除的目录和丢失数据的目录完全吻合。定位到脚本后，后面的问题都容易解决了。呵呵呵呵。

简单说一下，这个脚本最明显的问题就是，删除操作执行之前，没有判断是否压缩成功。

## 总结

凡是对数据操作的程序都要慎重。

***
2016-07-28 13:08:00 hzct
