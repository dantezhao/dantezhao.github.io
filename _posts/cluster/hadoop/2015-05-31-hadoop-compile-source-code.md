---
layout: post
title:  "Hadoop：Centos6.5（64bit）编译Hadoop2.5.1源码"
categories: 漫步云端
tags: Hadoop
---

* content
{:toc}


## 0.前言

Apache官网提供了Hadoop2.5.1已编译的程序，但是在Centos6.5上安装成功后，每当运行`hadoop fs ***`命令的时候总是会出现如下警告，虽说不影响运行结果，总是感觉影响心情。

因此搜集了一些资料，自己编译源码。比想象中的简单很多，只是在使用JDK的时候因为用了JDK1.8因此出现了一次错误，换成1.7就OK了，下面是安装记录。




```
WARN org.apache.hadoop.util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

## 1.编译环境

```
Centos6.5(64bit)      
jdk7      
ant1.9.4  
maven3.1.1  
findbugs3.0.0  
protobuf2.5.0  
hadoop2.5.1 源代码文件  
```

## 2.安装以上所需的所有软件
### 2.1yum可安装的软件
```
yum install svn ncurses-devel gcc* lzo-devel zlib-devel autoconf automake libtool cmake openssl-devel
```
### 2.2安装JDK

```
rpm -ivh jdk-7-linux-x64.rpm  
vim  /etc/profile  
export JAVA_HOME=/usr/java/jdk1.7.0  
export JRE_HOME=/usr/java/jdk1.7.0/jre  
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH  
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

### 2.3安装ant

```
tar zxvf apache-ant-1.9.4-bin.tar.gz  

vim /etc/profile  
export ANT_HOME=/usr/local/apache-ant-1.9.4  
export PATH=$PATH:$ANT_HOME/bin
```


### 2.4安装findbugs

```
tar zxvf findbugs-3.0.0.tar.gz  

vim /etc/profile  
export FINDBUGS_HOME=/usr/local/findbugs-3.0.0  
export PATH=$PATH:$FINDBUGS_HOME/bin  
```

### 2.5安装protobuf

```
tar zxvf protobuf-2.5.0.tar.gz  
cd protobuf-2.5.0  
./configure --prefix=/usr/local  
make && make install  
```

### 2.6安装maven

```
tar -zxvf apache-maven-3.1.1-bin.tar.gz  

vim /etc/peofile  
export M2_HOME=/usr/local/apache-maven-3.1.1  
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin  
```

## 3.编译hadoop源码

```
tar zxvf hadoop-2.5.1-src.tar.gz  
cd hadoop-2.5.1-src  
mvn package -Pdist,native,docs -DskipTests -Dtar
```

最后的文件就在`hadoop-2.5.1-src/hadoop-dist/target`中 。

***
2015-05-31 15:38:00 hzct
