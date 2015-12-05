---
layout: post
author: zhao
title:  Java工具：jstack
date:   2015-10-21 22:14:54
categories: Java
---

* content
{:toc}

##简介

###功能

jstack命令主要用于输出线程信息。

我们主要是看线程状态，在jstack中是可以得到遍历的。通常使用jstack -l <pid> 命令，如果未输出线程信息，则可以尝试使用-F参数强制输出。

###使用方法

~~~
[root@z1 week1]# jstack 
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
~~~


##例子

写一个死锁的例子，通过jstack查看线程的信息。


###程序

网上随便找了一个小的死锁例子。

程序中MyThread1和MyThread2都申请了两个Resource，他们之间存在一个死锁的情况。

~~~
public class DeadLock {  
    public static void main(String[] args) {  
        Resource r1= new Resource();  
        Resource r2= new Resource();  
        //每个线程都拥有r1,r2两个对象  
        Thread myTh1 = new MyThread1(r1,r2);  
        Thread myTh2 = new MyThread2(r1,r2);  
        myTh1.start();  
        myTh2.start();  
    }  
}  
  
class Resource{  
    private int i;  
}  
  
class MyThread1 extends Thread{  
    private Resource r1,r2;  
    public MyThread1(Resource r1, Resource r2) {  
        this.r1 = r1;  
        this.r2 = r2;  
    }  
  
    @Override  
    public void run() {  
        while(true){  
        //先获得r1的锁，再获得r2的锁     
        synchronized (r1) {  
            System.out.println("1号线程获取了r1的锁");  
            synchronized (r2) {  
                System.out.println("1号线程获取了r2的锁");  
            }  
        }  
        }  
    }  
      
}  
  
class MyThread2 extends Thread{  
    private Resource r1,r2;  
    public MyThread2(Resource r1, Resource r2) {  
        this.r1 = r1;  
        this.r2 = r2;  
    }  
  
    @Override  
    public void run() {
        while(true){  
        //先获得r2的锁，再获得r1的锁  
        synchronized (r2) {  
            System.out.println("2号线程获取了r2的锁");  
            synchronized (r1) {  
                System.out.println("2号线程获取了r1的锁");  
            }  
        }  
        }  
    }  
      
}  
~~~

###输出结果

可以看到，两个线程在获取资源的时候发生了死锁。

~~~
1号线程获取了r1的锁
2号线程获取了r2的锁
~~~

###线程信息

信息中有很多是java自己的进程，每个进程的状态也有显示。

在信息最后面的部分，可以看到MyThread1和MyThread2发生死锁的具体情况。

~~~
[root@z1 week1]# jstack -l 1382
2015-10-21 15:14:29
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007f72bc001000 nid=0x5ad waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"DestroyJavaVM" prio=10 tid=0x00007f72e0008800 nid=0x567 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Thread-1" prio=10 tid=0x00007f72e00a8000 nid=0x571 waiting for monitor entry [0x00007f72d78f7000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at MyThread2.run(DeadLock.java:53)
	- waiting to lock <0x00000000dd84e7e0> (a Resource)
	- locked <0x00000000dd84e7f0> (a Resource)

   Locked ownable synchronizers:
	- None

"Thread-0" prio=10 tid=0x00007f72e00a6000 nid=0x570 waiting for monitor entry [0x00007f72d79f8000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at MyThread1.run(DeadLock.java:31)
	- waiting to lock <0x00000000dd84e7f0> (a Resource)
	- locked <0x00000000dd84e7e0> (a Resource)

   Locked ownable synchronizers:
	- None

"Service Thread" daemon prio=10 tid=0x00007f72e008c800 nid=0x56e runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" daemon prio=10 tid=0x00007f72e008a000 nid=0x56d waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" daemon prio=10 tid=0x00007f72e0087800 nid=0x56c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" daemon prio=10 tid=0x00007f72e0085800 nid=0x56b runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" daemon prio=10 tid=0x00007f72e0064800 nid=0x56a in Object.wait() [0x00007f72d7ffe000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000000dd804858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000000dd804858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
	- None

"Reference Handler" daemon prio=10 tid=0x00007f72e0062800 nid=0x569 in Object.wait() [0x00007f72dc14d000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000000dd804470> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000000dd804470> (a java.lang.ref.Reference$Lock)

   Locked ownable synchronizers:
	- None

"VM Thread" prio=10 tid=0x00007f72e005e800 nid=0x568 runnable 

"VM Periodic Task Thread" prio=10 tid=0x00007f72e0097800 nid=0x56f waiting on condition 

JNI global references: 107


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f72d0003828 (object 0x00000000dd84e7e0, a Resource),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f72d00062c8 (object 0x00000000dd84e7f0, a Resource),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at MyThread2.run(DeadLock.java:53)
	- waiting to lock <0x00000000dd84e7e0> (a Resource)
	- locked <0x00000000dd84e7f0> (a Resource)
"Thread-0":
	at MyThread1.run(DeadLock.java:31)
	- waiting to lock <0x00000000dd84e7f0> (a Resource)
	- locked <0x00000000dd84e7e0> (a Resource)

Found 1 deadlock.
~~~






