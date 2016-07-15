---
layout: post
title:  "Linux：磁盘分区和挂载（parted和fdisk两种方式详细说明）"
categories: Linux
tags:  linux disk
---

* content
{:toc}

## parted和fdisk

parted命令可以划分单个分区大于2T的GPT格式的分区，也可以划分普通的MBR分区，fdisk命令对于大于2T的分区无法划分（大于2.2TB的存储空间用fdisk不支持，需要采用parted来分区），所以用fdisk无法看到parted划分的GPT格式的分区。





##parted和fdisk在操作上的区别

下面通过两个例子来说明。

环境：

- Centos6.5 64位操作系统（阿里云虚拟机）

- 双硬盘，一块以挂载，一块为空。所有的操作都在第二块硬盘上。

### parted分区操作步骤

先查看信息

```
[root@z3 ~]# parted -l
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  21.5GB  21.5GB  primary  ext4         boot


Error: /dev/xvdb: unrecognised disk label
```

开始分区

```
[root@z3 ~]# parted /dev/xvdb
GNU Parted 2.1
Using /dev/xvdb
Welcome to GNU Parted! Type 'help' to view a list of
commands.					                   
(parted) help					//使用帮助查看      
  align-check TYPE N                        check
        partition N for TYPE(min|opt) alignment
  check NUMBER                             do a simple
        check on the file system
  cp [FROM-DEVICE] FROM-NUMBER TO-NUMBER   copy file
        system to another partition
  help [COMMAND]                           print general
        help, or help on COMMAND
  mklabel,mktable LABEL-TYPE               create a new
        disklabel (partition table)
  mkfs NUMBER FS-TYPE                      make a
        FS-TYPE file system on partition NUMBER
  mkpart PART-TYPE [FS-TYPE] START END     make a
        partition
  mkpartfs PART-TYPE FS-TYPE START END     make a
        partition with a file system
  move NUMBER START END                    move
        partition NUMBER
  name NUMBER NAME                         name
        partition NUMBER as NAME
  print [devices|free|list,all|NUMBER]     display the
        partition table, available devices, free space,
        all found partitions, or a particular partition
  quit                                     exit program
  rescue START END                         rescue a lost
        partition near START and END
  resize NUMBER START END                  resize
        partition NUMBER and its file system
  rm NUMBER                                delete
        partition NUMBER
  select DEVICE                            choose the
        device to edit
  set NUMBER FLAG STATE                    change the
        FLAG on partition NUMBER
  toggle [NUMBER [FLAG]]                   toggle the
        state of FLAG on partition NUMBER
  unit UNIT                                set the
        default unit to UNIT
  version                                  display the
        version number and copyright information of GNU
        Parted
 (parted) mklabel					//选择格式					  
New disk label type? gpt
 (parted) mkpart					//选择分区的名称
 Partition name?  []? part1
 File system type?  [ext2]? 					//直接回车
Start? 1          					//从1开始
 End? 20G          					//选择20G大小
(parted) print    					//查看结果
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  20.0GB  20.0GB               part1

 (parted) mkpart   					//由于还有剩余空间，建立第二个分区
 Partition name?  []? part2
 File system type?  [ext2]? 
Start? 20.0GB     
End? 21.5GB       
 (parted) print    
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  20.0GB  20.0GB               part1
 2      20.0GB  21.5GB  1474MB               part2

(parted) quit     
 Information: You may need to update /etc/fstab.

//查看结果

[root@z3 ~]# parted -l
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  21.5GB  21.5GB  primary  ext4         boot


Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  20.0GB  20.0GB               part1
 2      20.0GB  21.5GB  1474MB               part2
```


然后进行格式化

```

[root@z3 ~]# mkfs.ext4 /dev/xvdb1
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1220608 inodes, 4882432 blocks
244121 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
149 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 39 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
[root@z3 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  2.1G   17G  12% /
tmpfs           938M     0  938M   0% /dev/shm
[root@z3 ~]# mkfs.ext4 /dev/xvdb2
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
90112 inodes, 359936 blocks
17996 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=369098752
11 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 33 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.

[root@z3 ~]# parted -l
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  21.5GB  21.5GB  primary  ext4         boot


Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  20.0GB  20.0GB  ext4         part1
 2      20.0GB  21.5GB  1474MB  ext4         part2

[root@z3 /]# mount /dev/xvdb1 /mnt
[root@z3 /]# mkdir /part2

//输入错误了，但是通过这次操作可以发现，同一个分区是可以挂载在两个目录下的。

[root@z3 /]# mount /dev/xvdb1 /part2


[root@z3 /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  2.1G   17G  12% /
tmpfs           938M     0  938M   0% /dev/shm
/dev/xvdb1       19G  172M   18G   1% /mnt
/dev/xvdb1       19G  172M   18G   1% /part2
[root@z3 /]# umount /part2
[root@z3 /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  2.1G   17G  12% /
tmpfs           938M     0  938M   0% /dev/shm
/dev/xvdb1       19G  172M   18G   1% /mnt
[root@z3 /]# mount /dev/xvdb2 /part2
[root@z3 /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  2.1G   17G  12% /
tmpfs           938M     0  938M   0% /dev/shm
/dev/xvdb1       19G  172M   18G   1% /mnt
/dev/xvdb2      1.4G   35M  1.3G   3% /part2

```

### fdisk分区操作步骤


```
//由于之前已经尝试分区过了，先删除，顺便记录一下删除惭怍。

[root@z2 ~]# fdisk -l

Disk /dev/xvda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00078f9c

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1   *           1        2611    20970496   83  Linux

Disk /dev/xvdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbc79a9bb

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1               1        2610    20964793+   5  Extended
/dev/xvdb5               1        2610    20964762   83  Linux


[root@z2 ~]# fdisk /dev/xvdb

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): d					//删除
Partition number (1-5): 1				//选择分区

Command (m for help): p		

Disk /dev/xvdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbc79a9bb

    Device Boot      Start         End      Blocks   Id  System

Command (m for help): n          		//新建
Command action
   e   extended
   p   primary partition (1-4)
p										//新建主分区
Partition number (1-4): 1				
First cylinder (1-2610, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-2610, default 2610): +19G

Command (m for help): n					
Command action
   e   extended
   p   primary partition (1-4)
e										//再新建一个扩展分区
Partition number (1-4): 2
First cylinder (2482-2610, default 2482): 				//直接默认起始位置
Using default value 2482
Last cylinder, +cylinders or +size{K,M,G} (2482-2610, default 2610): 
Using default value 2610

Command (m for help): n					
Command action	
   l   logical (5 or over)
   p   primary partition (1-4)
l									//在扩展分区上新建逻辑分区，直接使用所有的空间
First cylinder (2482-2610, default 2482): 
Using default value 2482
Last cylinder, +cylinders or +size{K,M,G} (2482-2610, default 2610): 
Using default value 2610

Command (m for help): p

Disk /dev/xvdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbc79a9bb

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1               1        2481    19928601   83  Linux
/dev/xvdb2            2482        2610     1036192+   5  Extended
/dev/xvdb5            2482        2610     1036161   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

[root@z2 ~]# fdisk -l

Disk /dev/xvda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00078f9c

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1   *           1        2611    20970496   83  Linux

Disk /dev/xvdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbc79a9bb

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1               1        2481    19928601   83  Linux
/dev/xvdb2            2482        2610     1036192+   5  Extended
/dev/xvdb5            2482        2610     1036161   83  Linux

```

格式化的时候需要注意，逻辑分区可能不能直接分区，网上的方法是使用partprobe，但是我一直不能成功，最后还是通过reboot解决。

```
[root@z2 ~]# mkfs.ext4 /dev/xvdb1
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1246032 inodes, 4982150 blocks
249107 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
153 block groups
32768 blocks per group, 32768 fragments per group
8144 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 28 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
[root@z2 ~]# mkfs.ext4 /dev/xvdb5
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
64768 inodes, 259040 blocks
12952 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8096 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 36 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
[root@z2 ~]# fdisk -l

Disk /dev/xvda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00078f9c

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1   *           1        2611    20970496   83  Linux

Disk /dev/xvdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbc79a9bb

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1               1        2481    19928601   83  Linux
/dev/xvdb2            2482        2610     1036192+   5  Extended
/dev/xvdb5            2482        2610     1036161   83  Linux



[root@z2 ~]# mount /dev/xvdb1 /mnt

[root@z2 ~]# mkdir /part2
[root@z2 ~]# mkfs.ext4 /dev/xvdb5
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
64768 inodes, 259040 blocks
12952 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8096 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 38 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
[root@z2 ~]# mount /dev/xvdb5 /part2
```

******
2015-08-14 22:14:54

