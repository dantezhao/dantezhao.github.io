---
layout: post
title:  "Hadoop多用户管理"
categories: 数据平台
tags: hadoop hdfs multi-tenant
---

* content
{:toc}

## 前言

最近有一些需求，就是需要在hadoop集群中实现多用户管理，因此在网上搜了很多的资料。其中有一种方法感觉还是比较可行，链接：http://cn.soulmachine.me/blog/20140206/大概方式是：

- 先新建一个用户test1，然后把hadoop的安装目录复制一份copy到这个用户test1的目录下
- 再赋一下权限，然后这个用户就可以向集群提交程序了。后来经过一些列的尝试，发现确实可以实现多用户管理，而且test1用户在hdfs中只能对自己的目录进行操作，但是毕竟资历太浅，只能隐隐约约感觉这种方法可以用，不明其原因。作为一名有轻度强迫症期望能先明白原理而后用的人来说，确实不太可取这种方法。

我也很好奇公司里面都是怎样使用hadoop多用户管理的，肯定会有比较好的解决方法，无奈自己不清楚，因此就用现有的linux和hadoop的一些知识尝试先实践一套方案，整个过程全部是自己摸摸索索，各种尝试出来的，有可能思路上也不是特别好。如有更好的方法，请指教。




## 一、背景

现有hadoop集群，安装目录在/home/hadoop/hadoop-2.5.1，hadoop的超级用户名就是hadoop，用户组也是hadoop
需要能够加入新的用户如：test1,test2,test3........

而且以后新加入用户的操作能比较简单。

这些test用户能够提交hadoop作业，而且能在hdfs中被指定的位置进行文件操作，他们能够看到本地hadoop安装目录的文件，但是只能看和执行，没有写和删除操作，不能启动和关闭集群，只能运行程序。

## 二、创建用户

```
创建新的用户，并且加入hadoop用户组中。
useradd -g hadoop test1
或者把一个已有用户加入hadoop用户组中。
usermod -a -G groups test1
```

## 三、本地目录赋权限

这一步的思路是，把具体的权限都赋给hadoop用户组，让用户组里的用户都能够执行下面的各种操作，因此以后新加入用户的时候只要把用户加入hadoop用户组就具备的相应的权限。

实现这一点，需要使用ACL权限管理机制，在此之前需要执行：（后来静下心来思考，情况简单的时候不用ACL就行，直接使用chmod赋权限就可以了）

```
mount -o remount,acl  /
此时执行mount就会看到： / 目录后面多了个acl，这样就可以使用acl来管理权限了。
```

把hadoop的安装目录（例如：/home/hadoop）权限设置为超级用户可读写执行，其他用户只能读和执行，即输入如下指令：

```
setfacl -R -m g:hadoop:rx hadoop
```

如果只是这样的话，在test用户提交作业的时候会提示本地的tmp文件没有写入权限，因此现在需要给设定的tmp文件赋予用户组hadoop写的权限
执行：

```
setfacl -R -m g:hadoop:rwx /home/hadoop/hadoop-2.5.1/tmp
```

## 四、hdfs权限管理

执行上面步骤后，就会发现，test1用户已经有了执行hadoop程序的权限，但是不能关集群，不能删除文件。

这时候就要考虑hdfs中的安全问题了，既然是test用户，我不希望test用户有太多的权限，特别是不能对某些特殊的文件进行操作（比如说/nlsde）,所以执行如下操作：

```
hdfs dfs -R -chmod 755 /
```

但是由于客户端在执行程序的时候会向tmp中写入数据，所有需要给hdfs中的/tmp赋予整个用户组的读写执行操作，因此需要执行：

```
hdfs dfs -R -chmod 777 /tmp
```

然后对于保密的文件夹，比如说/nlsde，我再执行：

```
hdfs dfs -R -chmod 700 /nlsde
```

这样的话，除了超级用户，就没有人能够访问这些文件了。


然后，我需要有一个文件夹/test，所有的test1，test2，等用户都能访问它，并且以后每个用户的工作环境也在这个目录里面：

```
hdfs dfs -R -chmod 777 /test
```

至此，基本环境已经成了，但是每个用户都应该有自己的私有空间，test1需要自己的内容不让别人看到，因此，我们需要给test1一个私有的文件夹。

```
hdfs dfs -mkdir /test/test1
hdfs dfs -chown test1 /test/test1
```

但是，这样还是不够的，其他用户照样可以访问这个文件夹，现在需要近一步设置，不让test2这些用户来访问这个文件夹。私有才是王道！

```
hdfs dfs -chmod 700 /test/test1
```

此时再看文件信息，其他用户已经不能再访问了。


## 五、添加新用户

现在添加新的用户测试，为了方面可以写成脚本比较方便：

```
sudo useradd -g hadoop test2
sudo passwd test2
hdfs dfs -mkdir /test/test2
hdfs dfs -chown test2 /test/test2
hdfs dfs -chmod 700 /test/test2
```

做些测试，使用test2，看看能不能访问test1的文件夹


Ok，至此hadoop的多用户管理已经基本能够满足我的一些需求了，但是这里面应该还有很多细节没有考虑到，需要在实际中才能发现问题。

我现在迫切地想知道的是，实际中的运维是怎样管理hadoop多用户管理的，是否也是我这种方式，或者是有其他更先进的方法？

通过这次尝试我就发现，自己在某些问题上还是会有一些偏执的，就像在解决多用户方面上，为了达到某个目标，在使用各种权限的操作，这种思路很难理清，能把这一套流程的思路正路下来需要花不少时间。

这篇博客给了我很大的启发：http://tiankefeng0520.iteye.com/blog/2029146

再次看hadoop的官网，发现了类似的方法，看来基本思路是对的：http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html

***
2014-10-20 14:43:00 bh xzl
