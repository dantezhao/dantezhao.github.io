---
layout: post
author: zhao
title: Docker：搭建开发环境（运行Eclipse等图形化界面程序）
modified: 2015-08-17
tags: [Docker]
image:
  feature: pic-7.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---


##基本说明

两个月前的时候自己提出想通过Docker来搭建开发环境（http://blog.csdn.net/zhaodedong/article/details/46549279），能方便地供实验室的其他同学使用。我所谓的开发环境没太复杂，只是能在一个docker镜像中运行Mysql、Jdk、Eclipse等基本的软件，但是Eclipse是需要能通过Docker启动可视化的界面。

最后这些功能的确能实现了，但是由于经常要在Windows中用PowerDesigner、Visio设计个数据库画个流程图什么的，Linux就不常作为桌面用了。因此对于Docker最初的设想就没有使用，时隔两个月，记录一下之前的学习过程。

##编写Dockerfile

我使用了Dockerfile来描述开发环境，下面是我写的一个只安装Eclipse的Dockerfile，诸如mysql，jdk什么的比较简单就不再写进来了。

{% highlight python linenos %}
FROM ubuntu:14.04

RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse" > /etc/apt/sources.list
RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse" >> /etc/apt/sources.list

RUN apt-get update && apt-get install -y libgtk2.0-0 libcanberra-gtk-module
RUN apt-get install -y eclipse

# Replace 1000 with your user / group id
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/developer && \
    echo "developer:x:${uid}:${gid}:Developer,,,:/home/developer:/bin/bash" >> /etc/passwd && \
    echo "developer:x:${uid}:" >> /etc/group && \
    echo "developer ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/developer && \
    chmod 0440 /etc/sudoers.d/developer && \
    chown ${uid}:${gid} -R /home/developer

USER developer
ENV HOME /home/developer
CMD /usr/bin/eclipse
{% endhighlight %}

##Docker Build

不多做描述，网上有很多教程讲Docker的基本操作。

{% highlight python linenos %}
docker build -t eclipse .
{% endhighlight %}

##启动可视化的Eclipse

里面有的参数我也不是特别熟悉，由于没有深入研究Docker的各个参数，现在也只是处于知其然而不知其所以然的境界。

{% highlight python linenos %}
docker run -ti --rm \
       -e DISPLAY=$DISPLAY \
       -v /tmp/.X11-unix:/tmp/.X11-unix \
       eclipse
{% endhighlight %}


##注意

中国的防火墙技术特别强大，但是网上的很多教程要不就是国外的要不就是国内拿过来随便翻译的，在这些教程里面经常会出现`RUN apt-get update` 这样的一个操作，这样是行不通的.......

解决方式就是我在Dockerfile里面写的，自己来修改Ubuntu的源，由于是在学校，我用的是清华的源，效果还是不错的。


##总结

Docker是个很好玩的东西，对于比较喜欢新技术的人来说是一个非常值得尝试的对象。但是有点遗憾，以后的学习和工作不一定能用到Docker了，因此再学习Docker也只能是自己的业余爱好中玩一玩了。

最初中想在Docker中搭建一个Hadoop集群的，但是发现如果固定了Docker的IP后，在Docker中安装Hadoop其实和在虚拟机中的操作没什么太大的区别，就没有再花时间具体的操作。

倒是看到了一些有趣的开源项目，直接编写的Dockerfile来配置和安装一个Hadoop集群，以后感兴趣的话可能会具体地尝试一下。

##参考

国内在Docker方面的资料还有所欠缺，至少在6月份我找资料的时候在各大博客网站中没有找到我需要的资料，谨列出对我帮助最大的几个。

http://www.tuicool.com/articles/ayIzI3

http://blog.zenika.com/index.php?post/2014/10/07/Setting-up-a-development-environment-using-Docker-and-Vagrant

http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/


