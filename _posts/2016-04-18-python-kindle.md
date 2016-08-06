---
layout: post
title:  "python实现Kindle笔记（标注）导出工具"
categories: 编程语言
tags:  python kindle
---

* content
{:toc}

## 前言

现在养成了使用python看杂书的习惯。前段时间顺手写了一个小程序，处理kindle的`My Clippings.txt`文件。

## 程序

直接上程序，发现还是有些小bug，写完我的csdn爬虫后找个周末专门调整一下。






```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import collections
import os
import re

#clips之间分隔符
BOUNDARY = u"==========\n"

#行分隔符
LINE_SEP = u"\n"

#文件输入路径
FILE_PATH = u"/home/dante/Documents/kindle/My Clippings.txt"

#文件输出目录
OUTPUT_DIR = u"/home/dante/Documents/kindle/output"



def get_sections(filename):
    """
    以BOUNDARY为分隔符，将文档切分
    """

    with open(filename, 'r') as f:
        content = f.read()
    content = content.replace(u'\ufeff', u'')

    return content.split(BOUNDARY)


def get_clip(section):
    """
    获取clip
    """
    clip = {}

    lines = [l for l in section.split(LINE_SEP) if l]

    if len(lines) != 3:
        return

    clip['book'] = lines[0]
    match = re.search(r'(\d+)-(\d+)', lines[1])
    if not match:
        return
    position = match.group(1)

    clip['position'] = int(position)
    clip['content'] = lines[2]

    return clip


def export_txt(clips):
    """
    生成文件
    每本书一个文件
    """
    print("into export_txt")
    for book in clips:
        lines = []
        for pos in sorted(clips[book]):
            #print("hah" + clips[book][pos])
            lines.append(clips[book][pos])
        print (lines)
        filename = os.path.join(OUTPUT_DIR, u"%s.txt" % book)
        #print (filename)
        with open(filename, 'w') as f:
            f.write("\n\n--\n\n".join(lines))


def checkdirectory():
    """
    检查输出目录是否存在，若不存在则生成
    """
    if not os.path.exists(OUTPUT_DIR):
        os.mkdir(OUTPUT_DIR)
    return

def main():

    checkdirectory()

    clips = collections.defaultdict(dict)


    # extract clips
    sections = get_sections(FILE_PATH)
    #print(sections)
    for section in sections:
       # print("into for cycle" + section)
        #print(len(section))
        clip = get_clip(section)
        #print(clip)
        if clip:
            clips[clip['book']][clip['position']] = clip['content']

    # remove key with empty value
    clips = {k: v for k, v in clips.items() if v}


    export_txt(clips)


if __name__ == '__main__':
    main()
```


## 总结

发点牢骚，和这个程序没关系，主要是在写爬虫时候。我是长记性了，以后再也不会一会跨操作系统写程序了。家里面的机器是linux，公司是windows，结果同样的程序，在公司就是有几个字段爬不下来，然后开了个虚拟机测了一才发现程序没问题，然后改着改着忘掉了，在windows下调程序了，能爬到所有字段了，又放到linux上，发现少字段了，我就是贱，以后不会再这样折腾了。

******
2016-04-18 22:44:00 hnds
