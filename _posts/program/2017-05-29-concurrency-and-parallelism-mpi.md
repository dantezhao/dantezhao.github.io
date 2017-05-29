---
layout: post
title:  "漫谈并发编程：用MPI进行分布式内存编程（入门篇）"
categories: 代码之熵
tags: C
---

* content
{:toc}


## 0x00 前言

> 本篇是MPI的入门教程，主要是为了简单地了解MPI的设计和基本用法，方便和现在的Hadoop、Spark做对比，并尝试理解它们之间在设计上有什么区别。

身处Hadoop、Spark这些优秀的分布式开发框架蓬勃发展的今天，老的分布式编程模型是否没有必要学习？这个很难回答，但是我更倾向于花一个下午的时候来学习和了解它。

关于并发和并行编程系列的文章请参考[文章集合](http://dantezhao.com)




### 文章结构

1. 举个最简单的例子，通过这个例子让大家对MPI有一个基本的理解。
2. 解释一些和MPI相关的概念。
3. 列举一些MPI的常用函数，以及基本用法
4. 通过两个例子详细说明MPI的用法

## 0x01 举个栗子

### 安装

建议在Ubuntu上安装，不过笔者尝试一下，报了各种错。正好Win10可以安装一个Linux的bash，就安装了一下，用起来和原生Linux没什么区别，挺方便。

一句搞定。

```
sudo apt-get install libcr-dev mpich2 mpich2-doc
```

### helloworld

MPI的c语言版helloworld。这是一个最简单的版本，相当于是每个进程都打印一下helloworld。

该例子中的一些方法以及概念在后面都会解释，而且会有两个比这个功能更全一点的例子来帮助理解。

```c
#include <mpi.h>
#include <stdio.h>

int main (int argc, char* argv[])
{
  int rank, size;

  MPI_Init (&argc, &argv);      /* starts MPI*/
  MPI_Comm_rank (MPI_COMM_WORLD, &rank);        /* get current process id*/
  MPI_Comm_size (MPI_COMM_WORLD, &size);        /* get number of processes*/
  printf( "Hello world from process %d of %d\n", rank, size );
  MPI_Finalize();
  return 0;
}
```


### 运行

先编译，如果有for循环的话，记得加上后面的参数。

```
mpicc mpi_hello.c -o hello -std=c99
```

再运行：

```
$ mpirun -np 5 ./hello
Hello world from process 0 of 5
Hello world from process 1 of 5
Hello world from process 2 of 5
Hello world from process 3 of 5
Hello world from process 4 of 5
```

### 总结

从上面的简单例子可以看出 一个MPI程序的框架结构可以用下图表示 把握了其结构之后，下面的主要任务就是掌握MPI提供的各种通信方法与手段。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/mpi_1.png)

### 安装时遇到的问题

来一个我在Ubuntu16.04下遇到的错误，实在不想解决这些乱七八糟的，就跳过了。

```
$sudo apt-get install libcr-dev mpich2 mpich2-doc
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package mpich2 is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  mpich:i386 mpich

Package mpich2-doc is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  mpich-doc

E: Package 'mpich2' has no installation candidate
E: Package 'mpich2-doc' has no installation candidate

```

## 0x02 基本概念


### 什么是MPI

对MPI的定义是多种多样的，但不外乎下面三个方面，它们限定了MPI的内涵和外延：

- MPI 是一个库，不是一门语言。MPI 提供库函数/过程供 C/C++/FORTRAN 调用。
- MPI 是一种标准或规范的代表，而不特指某一个对它的具体实现。
- MPI 是一种消息传递编程模型。**最终目的是服务于进程间通信这一目标** 。


### 名词和概念


**程序代码：**

这里的程序不是指以文件形式存在的源代码、可执行代码等，而是指为了完成一个计算任务而进行的一次运行过程。

**进程(Process)**

一个 MPI 并行程序由一组运行在相同或不同计算机 /计算节点上的进程或线程构成。为统一起见，我们将 MPI 程序中一个独立参与通信的个体称为一个进程。

**进程组：**

一个 MPI程序的全部进程集合的一个有序子集。进程组中每个进程都被赋予一个在改组中唯一的序号（rank），用于在该组中标识该进程。序号范围从 0 到进程数－1。

**通信器（communicator）:**

有时也译成通信子，是完成进程间通信的基本环境，它描述了一组可以互相通信的进程以及它们之间的联接关系等信息。MPI所有通信必须在某个通信器中进行。通信器分域内通信器（intracommunicator）和域间通信器（intercommunicator）两类，前者用于同一进程中进程间的通信，后者则用于分属不同进程的进程间的通信。

MPI 系统在一个 MPI 程序运行时会自动创建两个通信器：一个称为 MPI_COMM_WORLD，它包含 MPI 程序中所有进程，另一个称为MPI_COMM_SELF，它指单个进程自己所构成的通信器。

**序号（rank）：**

即进程的标识，是用来在一个进程组或一个通信器中标识一个进程。MPI 的进程由进程组/序号或通信器/序号唯一确定。

**消息（message）：**

MPI 程序中在进程间传递的数据。它由通信器、源地址、目的地址、消息标签和数据构成。

**通信（communication）：**

通信是指在进程之间进行消息的收发、同步等操作。


## 0x02 MPI核心接口

用过Hadoop的童鞋应该都记得经典的Map和Reduce接口，我们在写MR程序的时候主要就在写自己实现的Map和Reduce方法。

MPI比Hadoop需要关注的稍微多一点点。

**注意：** 这几个核心的接口还是要了解一下的。暂时可以看一眼跳过去，后面在看程序的时候回过头多对比一下就能记住了。

我们简单地理解一下这6个接口，其实可以分为3类：
1. 开始和结束MPI的接口：`MPI_Init`、 `MPI_Finalize`
2. 获取进程状态的接口：`MPI_Comm_rank`、`MPI_Comm_size`
3. 传输数据的接口：`MPI_Send`、`MPI_Recv`

关于传输数据的接口，可以看下图的理解。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/mpi_2.png)

### 1. `MPI_Init(&argc, &argv)`

初始化MPI执行环境，建立多个MPI进程之间的联系，为后续通信做准备。

### 2. `MPI_Comm_rank(communicator, &myid)`

用来标识各个MPI进程的，给出调用该函数的进程的进程号,返回整型的错误值。两个参数：MPI_Comm类型的通信域，标识参与计算的MPI进程组； &rank返回调用进程中的标识号。

### 3. `MPI_Comm_size(communicator, &numprocs)`

用来标识相应进程组中有多少个进程。

### 4. `MPI_Finalize()`

结束MPI执行环境。

### 5. `MPI_Send(buf,counter,datatype,dest,tag,comm)`

- buf：发送缓冲区的起始地址，可以是数组或结构指针；
- count：非负整数，发送的数据个数；
- datatype：发送数据的数据类型；
- dest：整型，目的的进程号；
- tag：整型，消息标志；comm：MPI进程组所在的通信域

含义:向通信域中的dest进程发送数据，数据存放在buf中，类型是datatype，个数是count，这个消息的标志是tag，用以和本进程向同一目的进程发送的其它消息区别开来。

### 6. `MPI_Recv(buf,count,datatype,source,tag,comm,status)`

- source:整型，接收数据的来源，即发送数据进程的进程号；
- status：MPI_Status结构指针，返回状态信息。


## 0x03 进阶版HelloWorld

在这里举第二个例子——一个进阶版的HelloWorld。不再像第一个例子那样简单地打印HelloWorld，在这个程序中，我们指派其中一个进程复杂输出，其它的进程向他发送要打印的消息。

### 程序

在这个程序中，为了方便理解我会注释大部分的代码。

注意注释。

```c

#include <stdio.h>
#include <string.h>  /* For strlen             */
//MPI相关的库
#include <mpi.h>     /* For MPI functions, etc */

const int MAX_STRING = 100;

int main(void) {
   char       greeting[MAX_STRING];  /* String storing message*/
   int        comm_sz;               //进程数
   int        my_rank;               //当前进程的进程号

   //初始化MPI
   MPI_Init(NULL, NULL);

   //获取进程的数量，并存入comm_sz变量中
   MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);

   //获取当前进程的进程号
   MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);

   // 进程号不为0的处理逻辑。
   // 在该程序中，进程号不为0的进程，只负责发数据给进程0。
   if (my_rank != 0) {
      //创建要发送的数据
      sprintf(greeting, "Greetings from process %d of %d!",
            my_rank, comm_sz);
      //发送数据给进程0
      MPI_Send(greeting, strlen(greeting)+1, MPI_CHAR, 0, 0,
            MPI_COMM_WORLD);
   }
   // 进程号为0的处理逻辑
   else {  
      // 打印进程0的数据
      printf("Hello! Greetings from process %d of %d!\n", my_rank, comm_sz);
      // 循环接收其它进程发送的数据，并打印。
      for (int q = 1; q < comm_sz; q++) {
         // 接收其它进程的数据
         MPI_Recv(greeting, MAX_STRING, MPI_CHAR, q,
            0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
         // 打印
         printf("%s\n", greeting);
      }
   }

   // 关闭MPI
   MPI_Finalize();

   return 0;
}  /* main */

```

### 运行

看一下运行结果。

第一条是我们的进程0打印的，其余的4条都是接收其它进程的数据。

```
$ mpicc mpi_hello.c -o hello  -std=c99
$ mpirun -np 5 ./hello
Hello! Greetings from process 0 of 5!
Greetings from process 1 of 5!
Greetings from process 2 of 5!
Greetings from process 3 of 5!
Greetings from process 4 of 5!

```

## 0x04 MPI实现梯形积分法

### 问题描述

用梯形积分法来估计函数 `y=f(x)` 的图像中，两条垂直线与x轴之间的区域大小。即下图（1）中阴影部分面积。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/mpi_3.png)

### 解法

如上图中的（b），基本思想就是，将x轴的区间划分为n个等长的子区间，然后估计每个子区间范围内的图形面积。最后相加即可。（当然了，这是估算的面积）

其中，梯形的面积如下：

```
梯形面积= h/2 * (f(xi) + f(xi+1))
```

其中高h是我们等分的一个区间值，`h=(b-a)/n`。

那么整个图形的面积如下：

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/mpi_4.png)

### 串行程序

根据上面的公式，我们可以得到一个串行版的程序：

```
h = (b-a)/n
approx = (f(a) + f(b)) /2.0
for(i=1;i<n-1;i++):
    x_i = a + i*h
    approx += f(x_i)
approx = h*approx
```

### MPI程序

整个MPI程序设计如下：

- 进程1~n, 负责各自的矩形面积
- 进程0，负责将所有矩形面积加起来求和

如下图

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/mpi_5.png)

对应到代码如下：

```
int main(void) {
   int my_rank, comm_sz, n = 1024, local_n;   
   double a = 0.0, b = 3.0, h, local_a, local_b;
   double local_int, total_int;
   int source;

   MPI_Init(NULL, NULL);
   MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);
   MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);

   // 区间大小，所有进程一样
   h = (b-a)/n;        
   local_n = n/comm_sz;  // 每个processor需要处理的梯形的数目

   /* Length of each process' interval of
    * integration = local_n*h.  So my interval
    * starts at: */
   local_a = a + my_rank*local_n*h;
   local_b = local_a + local_n*h;
   local_int = Trap(local_a, local_b, local_n, h);

   // 将所有的进程的结果相加
   if (my_rank != 0) {
      MPI_Send(&local_int, 1, MPI_DOUBLE, 0, 0,
            MPI_COMM_WORLD);
   }
   // 每个进程单独计算梯形面积
   else {
      total_int = local_int;
      for (source = 1; source < comm_sz; source++) {
         MPI_Recv(&local_int, 1, MPI_DOUBLE, source, 0,
            MPI_COMM_WORLD, MPI_STATUS_IGNORE);
         total_int += local_int;
      }
   }

   // 进程0打印结果
   if (my_rank == 0) {
      printf("With n = %d trapezoids, our estimate\n", n);
      printf("of the integral from %f to %f = %.15e\n",
          a, b, total_int);
   }
   MPI_Finalize();

   return 0;
} /*  main  */
```

Trap是梯形积分法的串行实现。供每一个processor调用。

```
double Trap(
      double left_endpt  /* in */,
      double right_endpt /* in */,
      int    trap_count  /* in */,
      double base_len    /* in */) {
   double estimate, x;
   int i;

   estimate = (f(left_endpt) + f(right_endpt))/2.0;
   for (i = 1; i <= trap_count-1; i++) {
      x = left_endpt + i*base_len;
      estimate += f(x);
   }
   estimate = estimate*base_len;

   return estimate;
}
```


结果

```
$mpirun -np 6 ./trap
With n = 1024 trapezoids, our estimate
of the integral from 0.000000 to 3.000000 = 8.894946975633502e+00

```



## 0xFF 总结

趁着端午假期的一个下午，把MPI做了一个小的总结。 程度不深，主要是了解MPI的一些基本特性。

暂时总结到这里，后续的工作和学习中如果再遇到了和MPI相关的知识点，再继续深入。

完整代码请看github地址。




## 参考

- 完整代码地址：https://github.com/dantezhao/concurrency_and_parallelism
- http://booksite.elsevier.com/9780123742605/
- https://jetcracker.wordpress.com/2012/03/01/how-to-install-mpi-in-ubuntu/
- http://math.ecnu.edu.cn/~jypan/Teaching/ParaComp/mpi_lect_jypan.pdf
- http://www.whigg.ac.cn/resource/superComputer/201010/P020101023579409136210.pdf

***
2017-05-29 20:31:00 wxxy
