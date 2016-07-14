---
layout: post
title:  "python实现csdn博客爬虫之1：整体设计思路"
categories: Python
tags:  python crawler 
---

* content
{:toc}

## 前言

很早之前就像自己写一些爬虫，把csdn、博客园这些博客平台的文章全部爬下来，把智联招聘、大街网、拉勾这些招聘网站的招聘信息爬下来，然后把github上面的各个项目信息爬下来。（当然，这些都只是想想）

后来发现自己执行力不够，这些都没开工。自从来了公司，相应公司“自我驱动”的号召，执行力稳步增长。赶着上周五的晚上的兴致，动手开始写了一个爬虫。下面记录一下整个纠结的过程。

**补充github项目地址：** [csdn_blog_spider](https://github.com/zhaodedong/csdn_blog_spider)


## 整体思路

整体设计思路很简单：

1. 先获取所有的用户ID（能获取多少是多少）
2. 获取每个用户的所有的博客id
3. 根据用户id和博客id拼接出博客url，然后爬下来。


## 设计过程

半年前想写的时候，其实我是直接在阶段四的，然后半年没写，再次捡起来的时候就想偷懒用框架了。

### 阶段一：使用scrapy？

最初是想用是个框架，比如scrapy，轻轻松松全部怕下来就OK了。后来想想，反正都是折腾，而且用框架的话也没啥意思，异常处理和算法什么的也都不需要自己考虑，还是自己折腾吧。再加上安装scrapy的时候报了好多次错误，彻底没积极性了。下面列举几个我遇到的错误，就会有人明白我为什么不想折腾了，折腾这个的功夫我自己都写完了，这些错误感觉和我的linux版本相关。

其实到后来，我碰见错误，连百度都懒了，直接在错误信息提示的那个文件后面加个`-dev`，然后安装一下就解决了，因此我放弃了使用scrapy。

错误1：

```
/tmp/xmlXPathInitmrZHMP.c:1:26: fatal error: libxml/xpath.h: 没有那个文件或目录
......
```
解决：
```
sudo apt-get install python-dev 
```


错误2：

```
src/lxml/includes/etree_defs.h:14:31: fatal error: libxml/xmlversion.h: 没有那个文件或目录
......
```

解决：

```
sudo apt-get install libxml2-dev
sudo ln -s /usr/include/libxml2/libxml   /usr/include/libxml
```

错误3：

```
src/lxml/includes/etree_defs.h:23:32: fatal error: libxslt/xsltconfig.h: 没有那个文件或目录
......
```

解决：
```
sudo apt-get install libxslt-dev
```

错误4：

```
/usr/bin/ld: cannot find -lz

collect2: error: ld returned 1 exit status

error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
......
```


解决：
```
sudo apt-get install zlib1g-dev
```

错误5：

```
No package 'libffi' found

Package libffi was not found in the pkg-config search path.

Perhaps you should add the directory containing `libffi.pc'

to the PKG_CONFIG_PATH environment variable

No package 'libffi' found

c/_cffi_backend.c:15:17: fatal error: ffi.h: 没有那个文件或目录

compilation terminated.

```

解决：

```
sudo apt-get install libffi-dev
```

### 阶段二：广度优先遍历博客？

第二种方式，就是写个简单的广度优先遍历喽，这个也不难嘛，一个queue+一个set就能搞定一个简单的小爬虫，这个代码写的比较简单。

整体思路就是写一个广度优先遍历，维护一个queue和一个set，然后就开始遍历了，只要记得加一个user-agent就好......

这个版本有个地方我不太喜欢，在写这个爬虫前我观察过csdn的博客url规则，就是`http://blog.csdn.net/用户名/article/details/博客id`，如果按照现在的爬虫，完全没有利用到这个规则啊，而且现在算法也写的比较烂，爬的时候感觉效率不是很高，暂时不能根据现在的场景优化太多目前这个版本的广度有限遍历算法。

### 阶段三：如何获取所有的用户id

那就换方案呗，反正前面的工作后面都可以用，比如说页面解析的代码。

这时候我暂定了一个设计方案，就是先抓取所有的用户ID，然后根据id获取用户的所有博客列表，然后再一次爬一个用户所有的博客，这样就Ok了。

第一个问题就是获取所有的用户id，之前的程序，小小地改一下正则表达式就可以用了，然后开始抓，我最初的思路比较受限制，想着只根据博客的地址来爬所有博客文章中的用户id，当时不知道怎么想的，这效率多低啊，周五晚上开始跑程序，第二天一看才抓了1.5w个用户id。当时我整个人心情就不好了。

和yyj聊过后，然后我就想歪招了！！！我在想，有没有方式脱一下csdn的库，不过鉴于目前的水平就算了。然后洗澡的时候突然想起来，之前csdn的库已经被别人脱过了，通过bing找到一个下载链接，解压后大概238M，600w条的用户记录，省了我一辈子的功夫去下载了。但是！！！但是，在用的时候，发现里面的用户名基本都是肥了，我写程序测了几万个，全部都没用，0粉丝，而且没开通博客，折腾一个小时发现没有一个用户名能用，好吧，我怀疑，这些账号是不是被csdn统一作废了？因为一看就是完全0活跃的用户。

好吧，我自己动手，想起来社交关系这块，然后我就进了`http://my.csdn.net/`里面，发现在每个用户主页里面可以获取到和他相关的一部分人的信息，数量从0-18个之间，然后又是轻轻地改一下正则表达式，就ok了。目前这个程序运行的还算不错，跑了一天，获取了总共45w的用户。

### 阶段四：抓取博客

抓取博客也需要分两步，先获取博客所有的id，然后再逐条抓取博文。这就比较简单了，处理好异常就行。

在这个阶段中，我发现，很多人的博客数量还是很多的，按照现在的用户数量，保守估计我能获取到4500W条博客，但是估计我是没那功夫跑这么多的数据了，因此有一些异常就直接跳过去，抓不到的就不抓了，反正博客那么多，随便抓点玩玩算了。

## 总结

这里面没贴代码，因为代码还一直在运行中，准备遇到问题了改改。后面贴代码。

在跑程序的时候遇到问题，爬博客的时候经常报500的错误，403错误通过user-agent可以搞定，这个问题我就一直很奇怪是哪的问题，莫非是csdn太弱了，一爬就崩？？？诡异，我直接try掉它了，少几篇就少几篇文章吧。

## 注意

小心磁盘，小心磁盘，小心磁盘。

两年前，我写程序不小心，坏过两块硬盘，这次我很注意，基本都是数据量到一定程度才写一次磁盘，中间可以再写一些机制，比如抓完十个用户的博客统一写一次磁盘，这时候，先把博客写进去后再把用户统一标成爬取，不然的话就不改变状态并且删除已经写入文件。

******
2016-04-18 23:35:00 hnds



