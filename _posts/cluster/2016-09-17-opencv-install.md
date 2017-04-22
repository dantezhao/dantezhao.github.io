---
layout: post
title:  "Ubuntu安装Opencv记录（附人脸识别和人眼识别例子）"
categories: 漫步云端
tags: Opencv
---

* content
{:toc}



## 安装步骤




### 环境

- Ubuntu 14.04 虚拟机
- Opencv 3.1.0


### 下载

- 官网：http://opencv.org/

- 国内的一个下载地址：http://blog.csdn.net/yanzi1225627/article/details/47668021

### 安装依赖

```
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg62-dev libtiff4-dev cmake libswscale-dev libjasper-dev
```

发现报了错，然后排除掉即可。

```
下列软件包有未满足的依赖关系：
 libtiff4-dev : 依赖: libtiff5-dev (> 4.0.3-6~) 但是它将不会被安装
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

执行下面这个安装命令就行

```
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg62-dev cmake libswscale-dev libjasper-dev
```

###  编译opencv

在官网下载opencv源码，解压后进入目录，执行（注意有个“.”，作为cmake的参数表示当前目录）

```
cmake .
```

然后

```
make
sudo make install
```

下面配置library，打开`/etc/ld.so.conf.d/opencv.conf`，在末尾加入`/usr/local/lib `   (有可能是个空文件，没关系)

然后

```
sudo ldconfig
```

然后编辑`/etc/bash.bashrc`加入

```
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH
```

至此，opencv安装配置完毕，下面开始测试

## 测试

网上的例子很多都没说清楚步骤。

### 识别人脸

新建一个目录，找到源码里面的sample中的例子，copy过来。`/opt/opencv-3.1.0/samples/cpp/facedetect.cpp`

然后新建一个CMakeLists.txt文件，写入下面内容。

```
cmake_minimum_required(VERSION 2.8)
project( FaceDetect )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_executable( FaceDetect facedetect.cpp )
target_link_libraries( FaceDetect ${OpenCV_LIBS} )
```

然后执行

```
cmake .
make
```

就可以运行了，在网上下载个图片瞄一眼。

```
./FaceDetect --cascade="/usr/local/share/OpenCV/haarcascades/haarcascade_frontalface_alt.xml" --scale=1.5 1361539897_5096.png
```

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/opencv-install-01.png)


![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/opencv-install-02.png)

### 人眼识别

就是换了一个xml文件......


```
./FaceDetect --cascade="/usr/local/share/OpenCV/haarcascades/haarcascade_eye.xml" --scale=1.5 1361539897_5096.png
```

![](http://zhaodedong.oss-cn-shanghai.aliyuncs.com/opencv-install-03.png)

## 总结

只是跑了一个例子，没有研究什么算法，也没有深入去看各种参数，权当配yyj先玩一玩。

## 参考

- http://www.cnblogs.com/jeakon/archive/2013/05/08/3066469.html
- http://www.tuicool.com/articles/nYJrYra

***
2016-09-17 22:39:00 rljp
