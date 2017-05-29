---
layout: post
title:  "漫谈并发和并行"
categories: 代码之熵
tags: Java C
---

* content
{:toc}

## 0x00 前言

比较担心自己最终有一天会陷入对各种工具的使用，而忽视了对一些基础知识的学习。因此，开始系列地整理一些知识。

本文关注并发和并行，虽说是漫谈，其实都是看书看知乎看各种文章，理论基本也都是凑出来的。我只是做了搬运工+自己的一丁点理解。




### 文章结构

- 概述，大致描述一下并发和并行的区别
- 摘录了两个关于并行和并发的区别，英语的那一段写的十分好。
- 列出来了4种并行的架构
- 放一个c++的多线程的例子

## 0x01 概述

> 并发是同一时间应对（dealing with）多件事情的能力！并行是同一时间动手做（doing）多件事情的能力！ ——Rob Pike

那么我们该怎么理解什么是并发？什么是并行呢？

- 并发：并发程序含有多个逻辑上的独立执行块，它们可以独立地并行执行，也可以穿行执行。注意**独立**这个词，它对我们理解这些概念很重要。
- 并行：并行程序解决问题的速度往往比串行程序快的多，因为其可以同时执行整个任务的多个部分，并行程序可能有多个独立执行块，也可能仅有一个？

换一个角度来看它们的差异：并发是问题域中的概念——要被设计成能够处理多个同时发生的事件；而并行则是方法域中的概念——通过将问题中的多个部分并行执行，来加速问题。

## 0x02 摘录

摘录一些其它渠道得来的回到。

### 摘录1（来自知乎：知乎用户）

- 并发（Concurrency）是同时处理很多事情（dealing with lots of things at once），并行（Parallelism）是同时执行很多事情（doing lots of things at once）；
- 二者有相关度，但并非同一个概念：并发可认为是一种逻辑结构的设计模式。你可以用并发的设计方式去设计模型，然后运行在一个单核系统上，通过系统动态地逻辑切换制造出并行的假象。此时，你的程序不是并行，但是是并发的。你可以将这种模型不加修改地运行在多核系统上，此时你的程序可以认为是并行。此处，并行更关注的是程序的执行（execution）；
- 在计算机中，我们通常会引入独立的运行实体来对并发模型的建模型，如：
    - 操作系统级别的进程和线程；
    - 编程语言内置的并发实体概念：
        - 如Golang 中的 goroutine（CSP 模型）；
        - Erlang 中的 process（Actor 模型）；
- 现实世界是并行的，人脑也是并行的。

### 摘录2（来自国外友人）

这个回答的十分好，很值得仔细看一遍。

The terms *concurrency* and *parallelism* are often used in relation to multithreaded programs. But what exactly does concurrency and parallelism mean, and are they the same terms or what?

The short answer is "no". They are not the same terms, although they appear quite similar on the surface. It also took me some time to finally find and understand the difference between concurrency and parallelism. Therefore I decided to add a text about concurrency vs. parallelism to this Java concurrency tutorial.

**Concurrency**

Concurrency means that an application is making progress on more than one task at the same time (concurrently). Well, if the computer only has one CPU the application may not make progress on more than one task at exactly the same time, but more than one task is being processed at a time inside the application. It does not completely finish one task before it begins the next.


**Parallelism**

Parallelism means that an application splits its tasks up into smaller subtasks which can be processed in parallel, for instance on multiple CPUs at the exact same time.


**Concurrency vs. Parallelism In Detail**

As you can see, concurrency is related to how an application handles multiple tasks it works on. An application may process one task at at time (sequentially) or work on multiple tasks at the same time (concurrently).

Parallelism on the other hand, is related to how an application handles each individual task. An application may process the task serially from start to end, or split the task up into subtasks which can be completed in parallel.

> As you can see, an application can be concurrent, but not parallel. This means that it processes more than one task at the same time, but the tasks are not broken down into subtasks.

> An application can also be parallel but not concurrent. This means that the application only works on one task at a time, and this task is broken down into subtasks which can be processed in parallel.

> Additionally, an application can be neither concurrent nor parallel. This means that it works on only one task at a time, and the task is never broken down into subtasks for parallel execution.

> Finally, an application can also be both concurrent and parallel, in that it both works on multiple tasks at the same time, and also breaks each task down into subtasks for parallel execution. However, some of the benefits of concurrency and parallelism may be lost in this scenario, as the CPUs in the computer are already kept reasonably busy with either concurrency or parallelism alone. Combining it may lead to only a small performance gain or even performance loss. Make sure you analyze and measure before you adopt a concurrent parallel model blindly.

## 0x03 并行架构

### 1.位级（bit-level）并行

为什么32位计算机比8位计算机运行速度更快？因为并行。对于两个32位数的加法，8位计算机必须进行多次8位计算，而32位计算机可以一步完成，即并行地处理32位数的4字节。计算机的发展经历了8位、16位、32位，现在正处于64位时代。然而由位升级带来的性能改 善是存在瓶颈的，这也正是短期内我们无法步入128位时代的原因。

### 2.指令级（instruction-level）并行

现代CPU的并行度很高，其中使用的技术包括流水线、乱序执行和猜测执行等。程序员通常可以不关心处理器内部并行的细节，因为尽管处理器内部的并行度很高，但是经过精心设计，从外部看上去所有处理都像是串行的。而这种“看上去像串行”的设计逐渐变得不适用。处理器的设计者们为单核提升速度变得越来越困难。

**进入多核时代，我们必须面对的情况是：无论是表面上还是实质上，指令都不再串行执行了。**

### 3.数据级（data-level）并行

并行地在大量数据上施加同一操作。这并不适合解决所有问题，但在适合的场景却可以大展身手。 图像处理就是一种适合进行数据级并行的场景。比如，为了增加图片亮度就需要增加每一个像素的亮度。现代GPU（图形处理器）也因图像处理的特点而演化成了极其强大的数据并行处理器。

### 4.任务级（task-level）并行

从程序员的角度来看，多处理器架构最明显的分类特征是其内存模型（共享内存模型或分布式内存模型）。

- 对于共享内存的多处理器系统，每个处理器都能访问整个内存，处理器之间的通信主要通过内存进行。
- 对于分布式内存的多处理器系统，每个处理器都有自己的内存，处理器之间的通信主要通过网络进行。

*通过内存通信比通过网络通信更简单更快速，所以用共享内存编程往往更容易。然而，当处理器个数逐渐增多，共享内存就会遭遇性能瓶颈——此时不得不转向分布式内存。如果要开发一个容错系统，就要使用多台计算机以规避硬件故障对系统的影响，此时也必须借助于分布式内存。*

## 0x04 举个栗子

前面都是理论，这就放一个极简单的c++的多线程的例子吧。程序这么简单就不再讲了。

```c
#include <pthread.h>
#include "stdio.h"
using namespace std;

#define NUM_THREADS 3

// 线程的运行函数
void* say_hello(void* args)
{
    printf ("Hello Dante! You're Great!\n");
}

int main()
{
    // 定义线程的 id 变量，多个变量使用数组
    pthread_t tids[NUM_THREADS];
    for(int i = 0; i < NUM_THREADS; ++i)
    {
        //参数依次是：创建的线程id，线程参数，调用的函数，传入的函数参数
        int ret = pthread_create(&tids[i], NULL, say_hello, NULL);
		printf ("Hello Dante! You're Gorgeous!\n");
        if (ret != 0)
        {
           printf ("pthread_create error: error_code=%d \n", ret);
        }
    }
    //等各个线程退出后，进程才结束，否则进程强制结束了，线程可能还没反应过来；
    pthread_exit(NULL);
}
```

运行结果如下，注意一下打印的结果，第二次打印的和其余的不一样？为什么不一样，就不补充了。

```
dante@DESKTOP-AE2RHL0:/mnt/d/workspace/c++$ ./a.out
Hello Dante! You're Great!
Hello Dante! You're Gorgeous!
Hello Dante! You're Gorgeous!
Hello Dante! You're Great!
Hello Dante! You're Gorgeous!
Hello Dante! You're Great!
```

## 0x05 总结

并发编程还是有很多要学的，而且不同语言对于并发编程的支持各有不同，后序我会对各个并发模型进行总结和整理，通过不同语言的示例来说明。


## 0xFF 参考

- **github地址：** https://github.com/dantezhao/concurrency_and_parallelism/blob/master/simple_cpp_example/hello_world.cc
- 知乎回答：https://www.zhihu.com/question/33515481/answer/135306366
- Jakob Jenkov：http://tutorials.jenkov.com/java-concurrency/concurrency-vs-parallelism.html
- 《七天七并发模型》


***
2017-04-04 15:55 wxxy
