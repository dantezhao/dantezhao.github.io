---
layout: post
title:  "chromebook install kali"
categories: Hacker
tags:  chrome google hacker
---

* content
{:toc}

## 0x00 前言

一千多块钱买的asus c200的chromebook，一千多块钱从美亚寄过来的，大概花了一个月的时间。

本来想把chrome os完全格了重新搞一个kali玩玩，但是u盘启动比较麻烦，还要刷bios，暂时放弃，继续用crouton来装，其实用着挺爽的，少了很多麻烦，而且在kali里面基本上什么功能都能用。

由于16G的硬盘太小，因此重新买了64的sd卡，格式化一下操作系统（挺方便，在设置里面一键搞定），重新装一下kali。





## 0x01 开发者模式

网上教程很多，不再重复。

## 0x02 crouton

github项目的主页，下载一个crouton即可。


```
https://github.com/dnschneid/crouton
```

### crouton用法

主要会使用下面的四个参数

```

$ sh ~/Downloads/crouton 

-r RELEASE  Name of the distribution release. Default: precise,
                or auto-detected if upgrading a chroot and -n is specified.
                Specify 'help' or 'list' to print out recognized releases.

-n NAME     Name of the chroot. Default is the release name.
                Cannot contain any slash (/).

-t TARGETS  Comma-separated list of environment targets to install.
                Specify 'help' or 'list' to print out potential targets.

-p PREFIX   The root directory in which to install the bin and chroot
                subdirectories and data.
```

### 查看release

```
$ sh ~/Downloads/crouton -r list
Recognized debian releases:
    potato* woody* sarge* etch* lenny* squeeze* wheezy jessie stretch sid
Recognized kali releases:
    kali* sana
Recognized ubuntu releases:
    warty* hoary* breezy* dapper* edgy* feisty* gutsy* hardy* intrepid* jaunty*
    karmic* lucid* maverick* natty* oneiric* precise quantal* raring* saucy*
    trusty utopic* vivid* wily* xenial*
Releases marked with * are unsupported, but may work with some effort.
```

## 0x03 挂载sd卡

sd卡挂载的目录有一个空格，装kali的时候，会因为这个空格报错。

```
chronos@localhost /media/removable/SD Card/kali $ df -h
Filesystem                                                    Size  Used Avail Use% Mounted on
/dev/mmcblk1p1                                                 60G   17M   60G   1% /media/removable/SD Card
```

重新挂载一个新目录。

```
chronos@localhost /media/removable $ sudo umount /dev/mmcblk1p1 
chronos@localhost /media/removable $ sudo mount /dev/mmcblk1p1 /media/removable/sd/
FUSE exfat 1.1.0
WARN: volume was not unmounted cleanly.
```
**注意：**

这样玩是不行的，重启后的目录还是以前的SD Card，比较坑的是chromebook没有给开fstab的权限，用root也写不进去内容。

好嘛，我老老实实不改名字了。

## 0x04 格式化sd卡

直接安装后发现sd的格式不对，我很好奇，chromebook格式化的文件系统为何不是ext的......

chromebook的mkfs使用体验极差，格式化64G的sd卡的时候各种的重启，无语死了，只有借用同事的ubuntu，记得格式化的时候加一个`-T largefile`参数 ,秒完，不再写命令了，太基本的操作了。

## 0x05 安装kali

安装就是使用crouton，添加好参数就行。


```
sudo sh -e ~/Downloads/crouton -r sana -t xfce -n kali -u -p /media/removable/SD\ Card

```
**注意1：** kali的版本要选用sana。

**注意2：**应该是安装到sd卡的问题，安装过程中出现好几次sd被卸载的情况，结果安重新装了好几遍才成功。

## 0x06 安装软件

### 全量安装

16G硬盘的就不要用这个方式了，保证地方不够，亲测。硬盘比较大的就好说了，全量装最开心。

```
sudo apt-get install kali-linux-full

```

### 按需安装

需要什么就安装什么了。比如nmap、ettercap这些东东。

```
sudo apt-get install ettercap-graphical
```

******
2016-04-01 13:00:00 hzct
******
