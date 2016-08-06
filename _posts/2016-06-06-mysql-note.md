---
layout: post
title:  "Mysql小记"
categories: Database
tags:  mysql
---

* content
{:toc}

## 前言

整理一下自己经常使用但是偶尔会记不住mysql的操作，免得每次都百度搜。

比较基础的就算了。





## Details

### 安装

```
安装：yum install -y mysql-server mysql mysql-devel
启动：service mysqld start
设置开机启动： chkconfig mysqld on
```

### 修改密码

#### 方法一
```
mysqladmin -u root password 'root'
```

#### 方法二

```
use mysql
update user set password=password('mylove') where user='root';
flush privilege;
```

### 开启远程登录

```
1.GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '密码' WITH GRANT OPTION;
2.FLUSH PRIVILEGES;
3.关闭防火墙，或者开放制定的3306端口
service iptables stop
```

### 远程登录

```
mysql –hIP地址 –u账号 –p密码
```
### 导入脚本

```
source /path/jiaoben.sql
```

### 备份数据库（导出脚本）


```
mysqldump -h IP -u 账号 -p'密码' 数据库名>jiaoben.sql
```


******
2016-06-06 12:00:01 hzct
