---
layout: post
author: zhao
title: Python：Mechanize模拟浏览器行为
modified: 2015-07-17
tags: [Python]
image:
  feature: pic-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

##使用Mechanize模拟浏览器行为

Python有许许多多有趣的模块，每当自己需要解决某个问题的时候，Python总能冒出来一两个让你惊喜的小玩意。比如说用于数值计算的**Numpy**（强大而方便的矩阵能力），用于数据分析的**Pandas**（和R语言有非常多相似的功能，在读写各种文件以及数据处理上会让人有种把excel、R、机器学习融合起来使用的感觉），用于爬虫内容提取的**BeautifulSoup**（点对点的精准数据获取，使用非常方便），以及最近正在使用的用于模拟浏览器登录的**Mechanize**。

##业务需求

既谈技术，先明需求，学习Mechanize的真实目的不方便描述，以下是学Mechanize带来的福利。

>现有一论坛，由于某种特殊原因需要定时发帖。比如一些校园内网的BBS，如果想浏览帖子，必须先登录。也就是说现在需要一个工具，能够登录该论坛，然后在相应的文本输入框中输入汉字，最后提交。

其实原理就是模拟浏览器和Server交互的一个过程，主要在于协议的一些理解。经过一些调研，有两种方案可选：Java的HttpClient和Python的Mechanize。两者都可实现模拟浏览器进行交互的一些功能，实现难度都不大。

HttpClient的方便之处在于文档比较全，还可以直接看源码，官网（https://hc.apache.org/index.html）还有各种小例子，有些可以直接使用。目前最新版本是4.5。

Mechanize使用比较简单，它保留许多与出色的 Expect 脚本相同的东西，它的使用过程，比如.select_form()、.submit()、.follow_link()等方法确实比较还原真实的“查找并发送”操作。遗憾的是文档并不想网上说的那么详细，官方给了几个例子，但是没有像Java那样的API，好多方法需要自己来摸索和看网上的例子，在使用的时候想查看所有的方法介绍比较麻烦。但是这不能阻挡我使用它的决心。Mechanize很久没更新了，目前版本是0.2.5。官网：http://wwwsearch.sourceforge.net/mechanize/

##Mechanize介绍

mechanize是对urllib2的部分功能的替换，能够更好的模拟浏览器行为，在Web访问控制方面做得更全面。它对protocol, cookie, redirection都做了比较好的支持，再结合beautifulsoup和re模块，可以非常有效的解析web页面。

###常用函数

.CookieJar()：设置cookie
.Browser()：打开浏览器
.addheaders()：User-Agent，用来欺骗服务器的
.open()：打开网页，按照官网描述可以打开任意网页，不仅限于http
.select_form()：选择表单的，选择表单的ID的时候需要注意。
.form[]：填写信息
.submit()：提交

##例子

###从百度搜索

比较简单，先获取表单信息，然后填入相应信息，提交即可，最后查看返回信息。

{% highlight python linenos %}
import sys
import mechanize

#Browser
br = mechanize.Browser()

#options
br.set_handle_equiv(True)
#br.set_handle_gzip(True)
br.set_handle_redirect(True)
br.set_handle_referer(True)
br.set_handle_robots(False)

#Follows refresh 0 but not hangs on refresh > 0
br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

br.set_debug_http(True)
br.set_debug_redirects(True)
br.set_debug_responses(True)

#欺骗行为
br.addheaders = [('User-agent', 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1')]

#上面的代码主要用于初始化设置，最好设置一下


# 打开百度
r = br.open('https://www.baidu.com/')
#获取百度的表单，从中找到输入汉字的位置
for f in br.forms():
    print f
    
br.select_form(nr = 0)

#搜索关键字“火车”
br.form['wd'] = "火车"
br.submit()
# 查看搜索结果
brr=br.response().read()
#是html代码，能看到火车的搜索结果
print brr
{% endhighlight %}

###登录某论坛，并发贴

比如http://examplehome.com论坛，需要在该论坛的http://examplehome.com/ID=001的帖子里面回复。大体流程如下：

 - 先登录http://examplehome.com/login界面，启用cookie记录，记录cookie信息。
 - 登录界面之后，cookie中已保存了登录信息，获取该cookie信息，再打开http://examplehome.com/ID=001，获取表单信息。
 - 在相应位置填入信息，提交，最后查看结果。
 

{% highlight python linenos %}
import sys
import mechanize

#cookie
cj = mechanize.CookieJar()

#Browser
br = mechanize.Browser()

#options
br.set_handle_equiv(True)
#br.set_handle_gzip(True)
br.set_handle_redirect(True)
br.set_handle_referer(True)
br.set_handle_robots(False)

#Follows refresh 0 but not hangs on refresh > 0
br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

#debugging?
br.set_debug_http(True)
br.set_debug_redirects(True)
br.set_debug_responses(True)

#User-Agent (this is cheating, ok?)
br.addheaders = [('User-agent', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.81 Safari/537.36')]

#启用cookie
br.set_cookiejar(cj)
br.open("http://examplehome.com/login")

#print br.title()
br.select_form(nr = 0)

#获取表单信息，我已知道信息，故注释掉
#for f in br.forms():
    #print f
    
#account info
br.form['username'] = "name"
br.form['password'] = 'pawd'

br.submit()

#登录成功，查看登录成功的后的信息
#br_response = br.response().read()

#print br_response

#经过上面步骤，cookie里已经保存的登录信息，下面直接使用cookie即可免密码登录了

#Browser
br2 = mechanize.Browser()
#options
br2.set_handle_equiv(True)
#br.set_handle_gzip(True)
br2.set_handle_redirect(True)
br2.set_handle_referer(True)
br2.set_handle_robots(False)

#Follows refresh 0 but not hangs on refresh > 0
br2.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

#debugging?
br2.set_debug_http(True)
br2.set_debug_redirects(True)
br2.set_debug_responses(True)

#User-Agent (this is cheating, ok?)
br2.addheaders = [('User-agent', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.81 Safari/537.36')]
#设置 cookie
br2.set_cookiejar(cj)

#打开帖子的链接
r = br2.open("http://examplehome.com/ID=001")

br2.select_form(nr = 3)

#获取表单的信息
#for f in br2.forms():
#    print f

#在发帖处，填写回复内容
br2.form['message'] = "我是使用程序自动发帖的~"

br2.submit()

#查看发帖后的结果
br2_response = br2.response().read()
print br2_response
{% endhighlight %}
