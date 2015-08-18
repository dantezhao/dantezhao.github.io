---
layout: post
author: zhao
title: Docker：初步使用
modified: 2015-08-17
tags: [Docker]
image:
  feature: pic-4.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---
##写在前面

学习Docker，官方文档必不可少，官网提供了比较好的文档支持以及一个交互型教程的帮助，建议最初的时候先以官网为主，出问题后再找一些博客和资料帮助解决。

 - 安装教程：https://docs.docker.com/installation/ubuntulinux/
 
 - 使用教程：https://docs.docker.com/userguide/
 
 - 交互教程：https://www.docker.com/gettingstarted 

首先强烈建议玩一遍官方的一个交互式命令行入门教程。甚至要多玩几遍，加深印象。

##安装

###安装环境：

 - OS：Ubuntu14.04（笔记本）
 - Docker：1.6.2
 
###安装记录

Ubuntu14.04安装Docker省却了更新内核等各种操作，可直接安装，按照官网的方式即可，只有一步，网上一些博客的安装方式和官网的方式是不一样的。自己选择。

{% highlight python linenos %}
wget -qO- https://get.docker.com/ | sh
{% endhighlight %}

OK，纯净系统安装，没报任何令人不愉快的错误，安装完成。

###验证

####查看版本：

{% highlight python linenos %}
sudo docker version
{% endhighlight %}

输出结果：

{% highlight python linenos %}
	Client version: 1.6.2
	Client API version: 1.18
	Go version (client): go1.4.2
	Git commit (client): 7c8fca2
	OS/Arch (client): linux/amd64
	Server version: 1.6.2
	Server API version: 1.18
	Go version (server): go1.4.2
	Git commit (server): 7c8fca2
	OS/Arch (server): linux/amd64
{% endhighlight %}

####查看信息

{% highlight python linenos %}
sudo docker info
{% endhighlight %}

输出信息：
（我是安装成功后写的文档，所以会有一些多余的内容，初次使用应该不会出现。另：无形中暴露了自己的机器信息......）

{% highlight python linenos %}
	Containers: 12
	Images: 25
	Storage Driver: aufs
	 Root Dir: /var/lib/docker/aufs
	 Backing Filesystem: extfs
	 Dirs: 49
	 Dirperm1 Supported: false
	Execution Driver: native-0.2
	Kernel Version: 3.13.0-32-generic
	Operating System: Ubuntu 14.04.1 LTS
	CPUs: 8
	Total Memory: 7.362 GiB
	Name: prairie
	ID: RID2:KUDU:ZGN7:S2EC:WMHQ:4OK2:7MUU:J4WZ:X35Z:YJW2:A3CF:F22X
	Username: ××××××
	Registry: [https://index.docker.io/v1/]
	WARNING: No swap limit support
{% endhighlight %}

###运行一个例子


{% highlight python linenos %}
sudo docker run hello-world
{% endhighlight %}


输出结果：

注意其中的`This message shows that your installation appears to be working correctly.` 证明docker安装成功了。

{% highlight python linenos %}
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
{% endhighlight %}

##权限设置

###可能（肯定）出现的错误

在使用非root用户登陆的时候，如果命令不加sudo，会有如下错误：

{% highlight python linenos %}
test@prairie:/root$ docker info
{% endhighlight %}

{% highlight python linenos %}
FATA[0000] Get http:///var/run/docker.sock/v1.18/info: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS? 
{% endhighlight %}

这是没有加sudo引起的权限的问题。

###去掉sudo

方法是，新建一个docker用户组，然后把现有用户添加进该组即可

{% highlight python linenos %}
sudo usermod -aG docker zhao（假设以zhao用户登陆） 
{% endhighlight %}


然后再执行命令就不会报错了。

{% highlight python linenos %}
zhao@prairie:~$ docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
{% endhighlight %}
