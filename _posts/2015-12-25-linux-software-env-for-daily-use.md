---
layout: post
author: zhao
title:  Linux：常用软件安装（输入法，快盘，chrome等）
date:   2015-12-25 12:14:54
categories: Java
---

* content
{:toc}

#简介

工作环境迁移到了Linux上面，除了开发环境，还需要一系列的日常软件，下面记录目前已经安装的各种日常软件。

##环境


Distributor ID:	Ubuntu

Description:	Ubuntu 14.04.1 LTS

Release:	14.04

Codename:	trusty


##安装软件列表

- sogou搜狗输入法
- WizNote为知笔记
- chrome浏览器
- kate文本编辑器
- openvpn
- jdk
- git
- svn
- kuaipan金山快盘

#安装

##sogou: 

安装搜狗输入法之前需要先安装fcitx

~~~
sudo apt-get update
sudo apt-get install fcitx
~~~

安装后，可以在“系统设置->语言支持”里面选择默认使用fcitx输入法框架

~~~
download sogou.deb:http://pinyin.sogou.com/linux/
sudo dpkg -i sogoupinyin.deb 
~~~

##wiznote

安装

~~~
sudo add-apt-repository ppa:wiznote-team
sudo apt-get update
sudo apt-get install wiznote
~~~

然后输入WizNote启动即可，注意大小写。

##chrome:

~~~
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb 
#如果出现安装缺少依赖的错误，之下下面语句
sudo apt-get install -f
~~~

##kate:
一个比较不错的文本编辑器，安装比较方便，可以和vim配合使用。至少写个博客什么的还是比较方便的。

~~~
sudo apt-get install kate
~~~
 
##openvpn:

openvpn可以直接下载。
~~~
sudo apt-get install openvpn
#把vpn的文件放入/etc/openvpn目录下
#启动
sudo openvpn /etc/openvpn/client.ovpn
~~~

##jdk

修改/etc/profile配置文件，添加环境变量
 
```
export
JAVA_HOME=/opt/jdk1.7.0
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```
然后验证
 
~~~
source /etc/profile
java -version
~~~

##git,svn

~~~
sudo apt-get install git subversion -y
~~~

##kuaipan

金山快盘是我目前发现的国内唯一支持多个操作系统的网盘，用起来还不错，至少同步盘的功能比别的强多了。

~~~
wget http://archive.ubuntukylin.com:10006/ubuntukylin/pool/main/k/kuaipan4uk/kuaipan4uk_2.0.0.5_amd64.deb
sudo dpkg -i kuaipan4uk_2.0.0.5_amd64.deb 
sudo apt-get install -f
~~~
