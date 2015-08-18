---
layout: post
author: zhao
title: Docker：一些思考
modified: 2015-08-17
tags: [Docker]
image:
  feature: pic-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---


##个人思考

现在有一种现象是Docker已经充斥了整个互联网行业，只要是互联网行业内的学生或者从业人员，或多或少都会和Docker扯上一定的联系，即使不用，也会主动去了解和学习。

Hadoop刚出现的时候也是这种情形（开源社区的异常活跃，各种网络教程的出现，大家蜂拥地尝试），这些都彰显出了Hadoop强大的生命力以及诱人使用场景。经过了这几年的沉淀，Hadoop已经日趋成熟，就我个人的感觉是Hadoop已经快成为一种普遍和必备的技能，凡数据相关，即谈Hadoop。

那么Docker呢？Docker能走多远？

学习Docker一部分原因是紧跟一下潮流（跟风？），另一部分是个人的一个小的实际需求。

##使用需求

>凡谈技术，先有需求。

我的初步设想是使用Docker来构建一个**可复用**的开发环境。

比如说我现在的开发环境是STS+JDK+MYSQL，在STS中需要一些jar包或者插件，比如Sping，Hibernate等，不同的项目成员可能会需要重复部署这套环境，而且在部署的过程中由于版本等原因还可能会出现各种问题。

那么我想做的就是能把我现在的工作环境做成一个Docker的模板，团队成员可以直接用，而不用花费过多的时间和精力配置环境。考虑到Docker如此轻量级的特性，在开发环境的使用上会比虚拟机方便和便捷很多。

##考虑方案

上述的需求Docker未必都能支撑，目前处于尝试阶段，就现有调研和使用结果暂定两种方案，具体实现形式需要做进一步的尝试：

 - **方案一**：使用Dockerfile，在DockerFile中配置虚拟机的各项内容，其他人直接使用这个Dockerfile文件。
	 
	 **问题**：
	 
	 - Dockerfile功能有限，比如jar的导入等功能实现起来可能会有困难。
	 
	 - 学习成本大
			
 - **方案二**：做成一个模板镜像，其他人直接copy使用
 
	**问题**：
	
	 - 不确定Docker是否支撑这种方式
	 
	 - 跨服务器使用镜像是否方便（比如我在自己的工作机器部署一个模板镜像，其他成员想在自己的工作机器上使用这个镜像是否方便）
