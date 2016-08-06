---
layout: post
title:  "python实现csdn博客爬虫之2：抓取所有用户名"
categories: 编程语言
tags:  python crawler
---

* content
{:toc}

## 前言

按照之前我的思路，第一步就是先获取所有的用户名。在下载了csd被黑客脱裤的600w数据并且没法使用后，只有自己写程序了。

下面这个程序细节没有怎么注意，反正能正常的运行，目前在电脑试过跑了三天还没问题，截止现在已经为了抓了60多w的用户名了。





## 程序

程序的整体思路很简单，就是一个简单的广度优先遍历，一个queue存放待爬的url，一个set()来存放已经爬过的url，一个set存放爬下来的用户名。平均没抓取100个用户名统一写入一次文件。代码有些地方还没有修缮，目前也就是能跑通功能的阶段。

```
import os
import re
import socket
import urllib.request
from bs4 import BeautifulSoup
from collections import deque

#根据url获取用户名
def get_username(url_):
    #pattern = re.compile('.*blog\.csdn\.net/([\w\-\_]*)/?')
    pattern = re.compile('.*csdn\.net/([\w\-\_]*)/?')
    # 使用Pattern匹配文本，获得匹配结果，无法匹配时将返回None
    match = pattern.match(url_)
    if match and match.group(1) != "":
        #print(match.group(1))
        return match.group(1)

#处理url
def process_url(url_):
    black = ["javascript", "css", "weibo", "github", "login", "help", "error", "account", "login", "baidu","comment","#"]
    #过滤特殊字符串
    for b in black:
        if(url_.find(b) != -1):
            return None
    if(url_.find("http://") == -1 and url_.find("https://") == -1):
        return (url_prefix + url_)
    else :
        if url_.find("my.csdn") == -1 :
            return None
        else:
            return (url_)

#初始化set，重启程序后读取之前的文件
def init_set(file_path):
    if os.path.isfile(file_path) == False:
        open(file_path,"w")
    set_tmp = set()
    file = open(file_path,"r")
    try:
        lines = file.readlines()
    except:
        file.close
    file.close
    for line in lines:
        set_tmp |= {line.strip()}
    return set_tmp

#从文件中读取现有的所有用户名并拼接成url形式
def init_queue(file_path):
    queue_tmp = deque()
    file = open(file_path,"r")
    try:
        lines = file.readlines()
    except:
        file.close
    file.close
    for line in lines:
        queue_tmp.append(url_prefix + line)
    return queue_tmp

#将set的内容，存入指定的文件中
def store_set(content, file_path, write_type):
    file = open(file_path, write_type)
    try:
        for t in content:
            file.write(t + "\n")
    except:
        file.close

start_url = "http://my.csdn.net/"
url_prefix = "http://my.csdn.net/"

username_path="username.txt"
visited_path="visited_url.txt"

socket.setdefaulttimeout(60)

queue = deque()
visited = init_set(visited_path)
name_set = init_set(username_path)

headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0'}

tmp_set = set()


queue.append(start_url)
cnt = 0

while queue:
    cnt += 1
    url = queue.popleft()  # 队首元素出队
    visited |= {url}  # 标记为已访问
    #每隔5000次，将所有访问过的url刷入文件一次。
    if(cnt % 2000 == 0):
        store_set(visited, visited_path, 'w')
    print('正在抓取第 ' + str(cnt)+  '个url：' + url)

    req = urllib.request.Request(url, headers=headers)


    # 避免程序异常中止, 用try..catch处理异常
    try:
        data = urllib.request.urlopen(req).read()
        data = data.decode('utf-8')
    except:
        continue

    soup = BeautifulSoup(data,'html.parser')


    for link in soup.find_all('a'):
        #先判断是否可能为空
        if link.get('href') == None:
            continue
        url = process_url(link.get('href'))
        if url not in visited and url != None :
            queue.append(url)
            username = get_username(url)
            if username not in name_set and username not in tmp_set and username != None:
                #读取100个用户后，统一写入文件中，并更新name_set
                print(username)
                tmp_set |= {username}

                if len(tmp_set) == 100 :
                    store_set(tmp_set, username_path,'a')
                    tmp_set = set()
                    name_set |= tmp_set

```

## 总结

好久不写程序，有些细节把握的不好了。加入tmp_set后，没有修改`if username not in name_set and username not in tmp_set and username != None:`这一句，结果导致在有一段时间，爬下来了很多的重复的用户名，后来做了一下修改就ok。

程序里面加了批量写入用户名的处理，因此多了很多的代码。


*邪恶一下，突然想起来一个问题，都说大部分的黑客都有一个自己的密码库，今天意识到，可以在网上找很多的被脱裤的情况，然后把里面的密码全部提取出来作为自己的密码库，以后破解的时候岂不是无敌了？？？*

******
2016-04-19 23:08:00 hzct


来源：
http://blog.csdn.net/zhaodedong
http://zhaodedong.leanote.com
http://zhaodedong.com
