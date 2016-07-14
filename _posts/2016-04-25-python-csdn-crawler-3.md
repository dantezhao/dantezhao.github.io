---
layout: post
title:  "python实现csdn博客爬虫之3：抓取所有博文"
categories: Python
tags:  python crawler 
---

* content
{:toc}
## 前言

这是之前运行的小程序，跑了几天抓了不少的文章，期间遇到不少的bug，改了改现在还算能正常跑。

程序的功能都有比较清楚的注释，一看就明白。

**整个程序的主要作用就是依次抓取每个用户的所有的博客正文以及博客相关信息。**


## 程序

```
#coding:utf-8
import os
import re
import time
import random
import socket
import requests
import urllib.request
from bs4 import BeautifulSoup
from collections import deque


def get_random_sleep() :
    time.sleep(random.randint(0,2))

#抓取一个用户有几页博客list
def get_page_count(username):
    print('开始抓取用户：' + username + '的博客list页数')
    code=requests.get(url_prefix + username ,headers=headers).status_code
    if code == 200:
        have_blog = True
    else:
        hava_blog = False
        return 0
    try:
        url = url_prefix + username + '/article/list'
        req = urllib.request.Request(url, headers=headers)
        data = urllib.request.urlopen(req).read()
        data = data.decode('utf-8')
        soup = BeautifulSoup(data)
        page_content = soup.find(class_ = 'pagelist').get_text().strip()
        pattern = re.compile(r'.*共([0-9]*)页.*')
        match = pattern.match(page_content)
        if match and match.group(1) != "":
            print('用户：' + username + '共有' + match.group(1) + '页博客list')
            return int(match.group(1))
        else:
            return 0
    except:
        return 0

#抓取一个页面的所有id，返回set
def get_article_id_per_page(username, page_id):
    print('开始抓取用户：' + username + '博客第' + str(page_id) + '页的所有文章id')
    #存储一个页面的所有id
    article_id_set=set()
    page_url = url_prefix + username + '/article/list/' + str(page_id)
    req = urllib.request.Request(page_url, headers=headers)
    try:
        data = urllib.request.urlopen(req).read()
        data = data.decode('utf-8')
        soup = BeautifulSoup(data)
        #获取并处理所有的id
        for link in soup.find_all('a'):
            #先判断是否可能为空
            if link.get('href') == None:
                continue
            pattern = re.compile(r'.*' + username + '/article/details/([0-9]*)')
            match = pattern.match(link.get('href'))
            if match and match.group(1) != "":
                article_id_set |= {match.group(1)}
            else:
                continue
        print('该用户：' + username + '博客第' + str(page_id) + '页共计' + str(len(article_id_set)) + '篇博客')
        return article_id_set
    except:
        return set()

#抓取一个用户的所有博客ID
def get_article_id_per_user(username):
    print('开始抓取用户：' + username + '的所有文章ID！')
    page_total = get_page_count(username)
    article_id_set = set()
    if page_total == 0:
        article_id_set |= get_article_id_per_page(username, 0)
    for i in range(1, page_total+1):
        get_random_sleep()
        article_id_set |= get_article_id_per_page(username, i)
    print('用户：' + username + '的所有文章ID获取完毕，共计 ' + str(len(article_id_set)) + '篇文章')
    return article_id_set

#获取文章正文
def get_blog_content(soup):
    try:
        content = soup.find_all(id='article_content')[0].get_text()
        content_str = str()
        for l in content.splitlines():
            if l != "\n" and l != "" and l != None and l != "\t":
                content_str += (l.strip() + ' ')
    except:
        return ''
    return content_str.strip()

#获取文章标题
def get_blog_title(soup):
    try:
        title = soup.find(class_ = "link_title").get_text().strip()
    except:
        return ''
    return title

#获取文章发布时间
def get_blog_postdate(soup):
    try:
        postdate = soup.find(class_ = 'article_r').find('span',{'class':'link_postdate'}).get_text().strip()
    except:
        return ''
    return postdate

#获取文章阅读量
def get_blog_view(soup):
    try:
        view = soup.find(class_ = 'article_r').find('span',{'class':'link_view'}).get_text().strip()
        pattern = re.compile('([0-9]*).*')
        match = pattern.match(view)
        if match and match.group(1) != "":
            return match.group(1)
        else:
            return ''
    except:
        return ''

#获取文章评论数
def get_blog_comments(soup):
    try:
        comments = soup.find(class_ = 'article_r').find('span',{'class':'link_comments'}).get_text().strip()
        pattern = re.compile('.*\(([0-9]*)\)')
        match = pattern.match(comments)
        if match and match.group(1) != "":
            return match.group(1)
        else:
            return ''
    except:
        return ''

#获取文章分类信息
def get_blog_categorys(soup):

    try:
        categorys = soup.find(class_ = 'similar_c_t').get_text().strip()
        category_str = str()
        for category in categorys.splitlines():
            if category != "\n" and category != "" and category != None and category != "\t":
                category_str += (category.split('（')[0] + ' ')
    except:
        return ''
    return category_str.strip()

#获取一篇博客的所有信息
def get_blog_details(username, article_id):
    print('正在抓取用户：' + username + ' 的 ' + article_id + ' 这篇文章！')
    content_details = str()
    url = 'http://blog.csdn.net/' + username + '/article/details/' + article_id

    #todo:这个地方经常500错误
    try:
        req = urllib.request.Request(url, headers=headers)
        data = urllib.request.urlopen(req).read()
        data = data.decode('utf-8')
        soup = BeautifulSoup(data)
        content_details = username + delimiter + article_id + delimiter + get_blog_title(soup) + delimiter + get_blog_postdate(soup) + delimiter + get_blog_categorys(soup) + delimiter + get_blog_view(soup) + delimiter + get_blog_comments(soup) + delimiter + get_blog_content(soup)
        # print(content_details)
        return content_details
    except urllib.error.HTTPError as http_e:
        print(http_e.code)
        print(http_e.read().decode("utf8"))
        return ''
    except http.client.IncompleteRead as read_e:
        print(read_e.code)
        print(read_e.read().decode("utf8"))
        return ''

#抓取一个用户的所有文章
def get_blog_per_user(username):
    print('-----------------------------------------------')
    print('开始执行用户：' + username + '的任务！')
    print('-----------------------------------------------')
    article_id_set = get_article_id_per_user(username)
    print('-----------------------------------------------')
    blog_list = list()
    print('开始抓取用户：' + username + '的所有文章正文！')
    for article_id in article_id_set:
        try:
            blog_list.append(get_blog_details(username,article_id))
        except urllib.error.HTTPError as e:
            print(e.code)
            print(e.read().decode("utf8"))
            continue
        get_random_sleep()
    return blog_list

#将set的内容，存入指定的文件中
def store_set(content, file_path, write_type):
    print('开始写入文件：' + file_path)
    file = open(file_path, write_type)
    try:
        for t in content:
            file.write(t + "\n")
    except:
        file.close
    file.close

#将list的内容，存入指定的文件中
def store_list(content, file_path, write_type):
    print('开始写入文件：' + file_path)
    file = open(file_path, write_type)
    try:
        for t in content:
            file.writelines(t + "\n")
    except:
        file.close
    file.close

#初始化set，重启程序后读取之前的文件
def init_set(file_path):
    if os.path.isfile(file_path) == False:
        open(file_path,"w")
    set_tmp = set()
    file = open(file_path,"r")
    try:
        lines = file.readlines()
        for line in lines:
            set_tmp |= {line.strip()}
    except:
        file.close
    file.close

    return set_tmp

#随机返回一个user-agent
def get_random_agent():
    user_agents = [
              'Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.11) Gecko/20071127 Firefox/2.0.0.11',
              'Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)',
              'Mozilla/5.0 (compatible; Konqueror/3.5; Linux) KHTML/3.5.5 (like Gecko) (Kubuntu)',
              'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.8.0.12) Gecko/20070731 Ubuntu/dapper-security Firefox/1.5.0.12',
              "Mozilla/5.0 (X11; Linux i686) AppleWebKit/535.7 (KHTML, like Gecko) Ubuntu/11.04 Chromium/16.0.912.77 Chrome/16.0.912.77 Safari/535.7",
              "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:10.0) Gecko/20100101 Firefox/10.0 ",
              ]
    return random.choice(user_agents)

if __name__ == '__main__':

    username_path = 'username.txt'
    user_parsed_path = "./user_parsed.txt"
    user_black_path = "./user_black.txt"

    url_prefix = "http://blog.csdn.net/"
    delimiter = '@$'
    #socket.setdefaulttimeout(60)


    #初始化一个爬取过的用户set

    print('爬虫开始运行！')
    print('-----------------------------------------------')
    print('初始化参数')
    print('-----------------------------------------------')
    user_parsed_set = init_set(user_parsed_path)
    user_parsed_set_tmp = set()
    #用户黑名单，即垃圾用户名和没有开启blog的用户
    user_black_set = init_set(user_black_path)
    #从总用户名中减去已经访问过的和黑名单中的
    username_set = init_set(username_path) - user_parsed_set - user_black_set
    print('目前共计用户数： ' + str(len(username_set)) + '个')

    blog_list_tmp = list()

    user_cnt = 0
    print('参数初始化完毕')
    print('-----------------------------------------------')
    for username in username_set:
        user_agent = get_random_agent()
        headers = {'User-Agent':user_agent}
        if username not in user_parsed_set_tmp and username not in user_parsed_set and username not in user_black_set:
            #抓取一个用户所有blog
            blog_list_per_user = get_blog_per_user(username)
            #将该用户所有blog添加入列表中
            blog_list_tmp.extend(blog_list_per_user)
            #将该用户放入已爬取列表中
            user_parsed_set_tmp |= {username}
            user_cnt += 1
        if user_cnt % 10 == 0 :
            #将tmp中的user_parse并入整体的set中，并追加到文件
            user_parsed_set |= user_parsed_set_tmp
            store_set(user_parsed_set_tmp, user_parsed_path, 'a')
            #将这一批的文章存入一个文件中
            user_blog_path = time.strftime('%Y-%m-%d %X',time.localtime(time.time())) + '.txt'
            # print(blog_list_tmp)
            print('进入写文件的流程')
            store_list(blog_list_tmp, user_blog_path, 'w')

            #清空tmp文件
            blog_list_tmp = list()
            user_parsed_set_tmp = set()
            user_cnt = 0

```

## 总结

这个小爬虫的第一版已经能跑了，其实我的本意是先这样抓些数据，然后自己开始各种玩，但是yyj嫌这样数据不全，现在需要更全的数据，比如用户的社交关系、用户个人信息和评论内容这些东东。目前一下子没有冲劲了，周末想了想了，如果需要那么完善的内容，我的爬虫已经不太容易满足了，维护起来太麻烦，而且我也不想这样再折腾太久了。因此决定下一步试试用scapy来做。

******
2016-04-25 23:18:00 hzct