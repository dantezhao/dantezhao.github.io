---
layout: post
title:  "Cloud Foundry：单机版安装"
categories: 漫步云端
tags: Cloudfoundry
---

* content
{:toc}

## 前言

CloudFoundry在安装过程中有相当大的一部分坑都会出在网络上，因此在安装过程中如果出现一些奇奇怪怪的问题可以先考虑一下是不是被墙了。

解决安装时的网络问题提供下面我们在实际中基本上都尝试过的几个思路：

- vpn，网上有免费的vpn也可以自己买一个，该方式不稳定。

- 翻墙，就不解释了，该方式不稳定。

- 买虚拟机，比如在AWS上买一个虚拟机，虚拟机的地址在国外或者在香港也可以。但是有一点需要注意，CF单节点是部署在虚拟机中的，也就是说你买的虚拟机也要能创建虚拟机，目前测试只有AWS可行。

- 后文会提到，比较笨的就是写一个脚本，失败了重复部署，方法低级，但是最后成功就是靠这个方法。





## 前期准备

单机版CF首先在物理机上安装Bosh-lite，vagrant，VirtualBox，随后通过vagrant VirtualBox 创建一个ubuntu虚拟机，再通过Bosh-lite创建一系列的Container来运行CF的各个组件。

### ruby环境安装

#### 安装相关库和编译环境

```
yum install -y build-essential openssl curl libcurl3-dev libreadline6 libreadline6-dev git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev libxslt-dev autoconf automake libtool imagemagick libmagickwand-dev libpcre3-dev libsqlite3-dev  zlib-devel
```

#### rbenv环境安装

```
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
type rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

#### ruby环境安装

首先列出可安装的版本，然后选择后进行下载编译
```
rbenv install -l
rbenv install 1.9.3-p448
```

#### 设置ruby版本

由于网络原因，在安装ruby相关的一些组件的时候可能要改变成淘宝的镜像。

设置当前使用的ruby版本并将gem的源改为淘宝镜像。

```
rbenv global 1.9.3-p448
rbenv rehash
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
```

#### 验证安装是否成功

```
ruby -v
```

### 安装virtualbox、vagrant

分别下载，安装即可。 验证安装是否成功：

```
vagrant -v
VBoxManage --version。
```

### 安装Bosh lite

#### 安装bosh lite

```
gem install bosh_cli
```

### 获取”安装包“

```
mkdir workspace
cd ~/workspace
git clone https://github.com/cloudfoundry/bosh-lite
```

### 安装启动所需虚拟机

（国内网络情况问题，会等待较长时间，Centos有时会遇到内核版本与VirtualBox兼容问题）此时建议更改Vagrantfile文件来增加虚拟机的配额

```
vagrant up --provider=virtualbox
```

注意：此过程由于网速问题会非常慢，建议按照下面步骤执行，先下载

```
(https://atlas.hashicorp.com/cloudfoundry/boxes/bosh-lite/versions/9000.56.0/providers/virtualbox.box)
```

#### 添加vagrant box

手动本地添加vagrant box
```
vagrant box add --name virtualbox ./virtualbox.box
```

创建成功后可通过vagrant ssh登录，后用ssh 直接登录密码（vagrant）

### 添加路由规则

添加路由规则至物理机

在workspace/bosh-lite/bin/下执行

```
./add-route
```

至此前期准备工作完成


## 安装CF组件

### 登录

登录至虚拟机内，选择bosh target

```
bosh target 192.168.50.4 lite
bosh login
```

### 获取安装包

从github上获取安装包
```
cd ~/workspace
git clone https://github.com/cloudfoundry/bosh-lite
git clone https://github.com/cloudfoundry/cf-release.git
```

其目录结构如下：

```
└── workspace
	    ├── bosh-lite
	    ├── cf-mysql-release
	    ├── cf-release
	    └── go
```

### 安装CF

#### 自动化安装

自动安装只需执行`/home/vagrant/workspace/bosh-lite/bin/provision_cf`即可。

但是国内网速不但会很慢而且往往会掉线，这里建议写监听进程脚本来执行，最初我们下载了5天才安装完成......

监听脚本写起来比较简单，就是写一个小脚本，判断安装是否完成，没有顺利完成的话重新执行脚本，限于脚本写的太烂（用着倒也正常）就不贴出来了。

#### 手动安装

当二次部署时建议使用手动的部署方式，这样可以大大提升速度。

注意：**下载后的CF安装文件会以隐藏目录的方式，存放于`~/.bosh`目录下，下载后可用于手动反复安装**

### 上传Stemcell

```
 bosh upload stemcell bosh-stemcell-2776-warden-boshlite-ubuntu-trusty-go_agent.tgz
```

stemcell是cloud foundry的镜像模板，在单节点中是warden容器模板，在集群版中是linux操作系统的虚拟机，这里的stemcell文件建议在bosh官网下载，国内的网速会很慢

### 上传cf release



记住之前所下载的cf release 我这里是cf-217.yml，CF会经常更新。

确认cf-217.yml为当前安装版本。

```
 bosh upload release releases/cf-217.yml
```

### 创建cf-manifest.yml

#### 安装spiff

需要先安装spiff，执行以下脚本

```
/home/vagrant/workspace/bosh-lite/bin/make_manifest_spiff
```

此时会在`/home/vagrant/workspace/bosh-lite/manifests`路径下生成并指定cf-manifest.yml部署文件，若没指定则可手动指定

```
bosh deployment /home/vagrant/workspace/bosh-lite/manifests/cf-manifest.yml
```

### 执行部署

以上准备就绪后则可执行

```
bosh deploy
```

开始部署，一般会20分钟左右

### 查看部署情况

```
bosh vms
```

验证是否安装成功，若每个Container都为running则无异常

#### 可登录使用验证

```
cf api --skip-ssl-validation https://api.10.244.0.34.xip.io
cf auth admin admin
cf create-org test-org
cf target -o test-org
cf create-space test-space
cf target -s test-space
```

***
2015-09-10 17:11:30 bh xzl
