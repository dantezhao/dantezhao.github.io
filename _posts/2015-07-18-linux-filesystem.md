---
layout: post
author: zhao
title: Linux：文件系统
modified: 2015-08-18
tags: [Linux]
image:
  feature: pic-19.jpg
---

##文件系统特性

较新的操作系统的文件数据除了文件实际内容外， 通常含有非常多的属性，例如 Linux 操作系统的文件权限(rwx)与文件属性(拥有者、群组、时间参数等)。 文件系统通常会将这两部份的数据分别存放在不同的区块，权限与属性放置到 inode 中，至于实际数据则放置到 data block 区块中。 另外，还有一个超级区块(superblock) 会记录整个文件系统的整体信息，包括 inode 与block 的总量、使用量、剩余量等。

##索引式文件系统

假设某一个文件的属性与权限数据是放置到 inode 4 号(下图较小方格内)，而这个 inode 记录了文件数据的实际放置点为 2, 7, 13, 15这四个 block 号码，此时我们的操作系统就能够据此来排列磁盘的阅读顺序，可以一口气将四个 block 内容读出来，这种数据访问的方式称为索引式文件系统。

<figure class="half">
	<a href="/images/blog/index-filesystem.png"><img style="float:middle" src="/images/blog/index-filesystem.png"></a>
</figure>

##Ext2文件系统

当文件达到非常大的时候inode 与block放在一起是不明智的，因此Ext2区分多个组来管理。

<figure class="half">
	<a href="/images/blog/filesystem.png"><img style="float:middle" src="/images/blog/filesystem.png"></a>
</figure>

###Boot Sector

- 启动扇区，设备的第一个扇区
- 放置开机管理程序，加载并转让处理器控制权给操作系统
- 由主引导记录（MBR，Main Boot Recode，是一段程序）、磁盘分区表（DPT， Disk Partition Table）和硬盘有效标志（MN，Magic Number）组成
- 在系统启动过程中充当重要角色：
 - BIOS ->MBR（扫描分区表DPT）->启动操作系统

<figure class="half">
	<a href="/images/blog/boot-sector.png"><img style="float:middle" src="/images/blog/boot-sector.png"></a>
</figure>

###Superblock

- 记录的信息主要有:
 - block与inode 的总量
 - 未使用与已使用的inode/block数量
 - block与inode的大小(block为1,2,4K,inode为128 bytes)
 - 文件系统的挂载时间，最近一次写入数据的时间，最近一次检验磁盘(fsck)的时间等文件系统的相关信息

<figure class="half">
	<a href="/images/blog/superblock.png"><img style="float:middle" src="/images/blog/superblock.png"></a>
</figure>

###Filesystem Description(文件系统描述说明)

- 描述每个block group的开始与结束的block号码
- 说明每个区段(superblock,区块对应表,inodemap,data block)的位置

<figure class="half">
	<a href="/images/blog/filesystem-desc.png"><img style="float:middle" src="/images/blog/filesystem-desc.png"></a>
</figure>

###block bitmap（区块对应表）

- 新增文件时总会用到block，使用空的block来记录新文件的数据。
- block bitmap当中可以知道哪些block是空的,因此我们的系统就能够快速找到可以使用的空间来处置文件
- 删除文件时,文件原本占用的block号码就得要释放出来, blockbitmap就做相应记录（在block bitmap当中相对应的block号码标志修改成为未使用）。

<figure class="half">
	<a href="/images/blog/block-bitmap.png"><img style="float:middle" src="/images/blog/block-bitmap.png"></a>
</figure>

###inode bitmap

- 功能与block bitmap是类似,block bitmap记录的是使用和未使用的block号码, inode bitmap则是记录使用和未使用的inode号码
- 什么是inode，看下一张

<figure class="half">
	<a href="/images/blog/inode-bitmap.png"><img style="float:middle" src="/images/blog/inode-bitmap.png"></a>
</figure>

###inode table

- inode，记录文件的属性以及该文件实际数据放置在哪些block
- 该文件的存取模式(read/write/excute)
- 该文件的拥有者与群组(owner/group)
- 该文件的容量
- 该文件建立或状态改变的时间(ctime)
- 最近一次的读取时间(atime)
- 最近修改的时间(mtime)
- 定义文件特性的标志(flag)
- 该文件真正内容的指向(pointer)
- 每个inode大小均固定为128bytes
- 每个文件都仅会占用一个inode而已

<figure class="half">
	<a href="/images/blog/inode-table.png"><img style="float:middle" src="/images/blog/inode-table.png"></a>
</figure>

- 操作系统查找一个文件的过程
- 先找到inode number
- 根据inode number找到相应的inode table
- 从inode table的pointer中，定位到文件内容存储于什么block中

<figure class="half">
	<a href="/images/blog/inode-table-cont.png"><img style="float:middle" src="/images/blog/inode-table-cont.png"></a>
</figure>

###datablock

- data block是用来放置文件内容数据地方
- 在Ext2文件系统中所支持block大小有1K,2K及4K三种。
- 格式化时block的大小就固定了,且每个block都有编号,以方便inode记录。
- 由于block大小的差异,导致文件系统能够支持的最大磁盘容量和最大单一文件容量并不相同

| Block大小 | 1KB | 2KB | 4KB |
| 最大单一文件限制 | 16GB | 256GB | 2TB |
| 最大文件系统总容量 | 2TB | 8TB | 16TB |

####block限制

- 基本上block的大小与数量在格式化完就不能够改变(除非重新格式化);
- 每个block内最多只能够放置一个文件的数据;
- 如果文件大于block的大小,则一个文件会占用多个block数量;
- 如果文件小于block,则该block的剩余容量就不能够再被使用了,磁盘空间会浪费

<figure class="half">
	<a href="/images/blog/block.png"><img style="float:middle" src="/images/blog/block.png"></a>
</figure>