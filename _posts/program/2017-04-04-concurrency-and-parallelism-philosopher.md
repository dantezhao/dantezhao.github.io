---
layout: post
title:  "漫谈线程与锁：死磕哲学家进餐问题"
categories: 代码之熵
tags: Java
---

* content
{:toc}


## 0x00 前言

线程与锁可以说是并发领域中经典也是应用最广的模型，尽管它有很多众所周知的缺点，但是它依然是开发并发软件的首选技术。

线程与锁模型的缺点有很多，比如竞态条件、外星方法、死锁等。我们这里就死磕一下其中的死锁问题。




### 哲学家进餐问题

哲学家进餐问题是描述死锁最经典的问题，我们后续整个文章都会以此为出发点来讨论，现在先列出来哲学家进餐的问题描述。

> 问题场景是五个哲学家围绕一个圆桌就做，桌上摆着五只（不是五双）筷子。哲学家的状态可能是“思考”或者“饥饿”。如果饥饿，哲学家就将拿起他两边的筷子并就餐一段时间。就餐结束，哲学家就会放回筷子。

### 文章组织

本文主要是讲哲学家进餐问题，因此有必要先回顾一些锁的知识，然后会通过四个版本的程序来解决哲学家进餐问题，最后一个简单的总结。 在这里先说明四个版本的程序有什么区别：

- 经典的内置锁解决方案。
- 同样是内置锁，但是我们加上了全局变量来解决死锁问题。
- 使用ReentrantLock来替代内置锁，同时通过超时取消的机制来解决死锁。
- 在ReentrantLock的基础上引入条件变量，更优雅的解决问题。

每个版本的程序都会有完整的代码实现，并且有每种方式的优缺点，以及和其它方式的区别。

## 0x01 锁的基本概念

### 死锁

死锁是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

**死锁发生的条件**：

- 互斥条件：线程对资源的访问是排他性的，如果一个线程对占用了某资源，那么其他线程必须处于等待状态，直到资源被释放。
- 请求和保持条件：线程T1至少已经保持了一个资源R1占用,但又提出对另一个资源R2请求，而此时，资源R2被其他线程T2占用，于是该线程T1也必须等待，但又对自己保持的资源R1不释放。
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能被其他线程剥夺，只能在使用完以后由自己释放。
- 环路等待条件：在死锁发生时，必然存在一个“进程-资源环形链”，即：{p0,p1,p2,...pn},进程p0（或线程）等待p1占用的资源，p1等待p2占用的资源，pn等待p0占用的资源。（最直观的理解是，p0等待p1占用的资源，而p1而在等待p0占用的资源，于是两个进程就相互等待）

### 活锁

活锁是指线程1可以使用资源，但它很礼貌，让其他线程先使用资源，线程2也可以使用资源，但它很绅士，也让其他线程先使用资源。这样你让我，我让你，最后两个线程都无法使用资源。

### 活锁和死锁的区别

活锁和死锁的区别在于，处于活锁的实体是在不断的改变状态，所谓的“活”，而处于死锁的实体表现为等待；活锁有可能自行解开，死锁则不能。


## 0x02 版本1：内置锁

在第一个实现版本中，我们使用synchronized。这种方法效率比较低，而且十分容易产生死锁的现象。但是它仍不失为一种说明问题的方法。

整体代码分三部分：Chopstick类、Philosopher类和DiningPhilosophers类，其中DiningPhilosophers类是场景类，代表我们了进餐逻辑。

### 代码清单 Chopstick类

```java
/**
* 哲学家用的筷子类
*/
class Chopstick {
	private int id;
	public Chopstick(int id) { this.id = id; }
  public int getId() { return id; }
}
```

### 代码清单 Philosopher类

```java
/**
* 哲学家类
*/
class Philosopher extends Thread {
  private Chopstick left, right;
  private Random random;
  private int thinkCount;

  public Philosopher(Chopstick left, Chopstick right) {
    this.left = left; this.right = right;
    random = new Random();
  }

  public void run() {
    try {
      while(true) {
        ++thinkCount;
        if (thinkCount % 10 == 0)
          System.out.println("Philosopher " + this + " has thought " + thinkCount + " times");
        Thread.sleep(random.nextInt(10));     // Think for a while
        synchronized(left) {                    // Grab left chopstick
          synchronized(right) {                 // Grab right chopstick
            Thread.sleep(random.nextInt(10)); // Eat for a while
          }
        }
      }
    } catch(InterruptedException e) {}
  }
}
```

### 代码清单 DiningPhilosophers类

```java
/**
* 哲学家进餐的主类
*/
public class DiningPhilosophers {

  public static void main(String[] args) throws InterruptedException {
    //five philosophers
    Philosopher[] philosophers = new Philosopher[5];
    //five chopsticks
    Chopstick[] chopsticks = new Chopstick[5];

    //init philosophers and chopsticks
    for (int i = 0; i < 5; ++i)
      chopsticks[i] = new Chopstick(i);
    for (int i = 0; i < 5; ++i) {
      philosophers[i] = new Philosopher(chopsticks[i], chopsticks[(i + 1) % 5]);
      philosophers[i].start();
    }
    for (int i = 0; i < 5; ++i)
      philosophers[i].join();
  }
}
```

### 运行分析

接着我们运行我们的程序来观察一下哲学家进餐的情况。下面是运行的结果我们可以看出来，这些哲学家可以愉快地运行很长时间，直到某个时刻所有一切都突然停下来了。看下面的打印输出，可以看到代码停止了运行。

我们稍作分析，就能够知道发生了什么，如果在某个时间点，所有的哲学家同时决定用餐，都拿起左手的筷子，那么就无法进行下去。**所有人都持有只筷子，等待右边的人放下筷子。这时死锁出现了。**

下面通过jstack看一下死锁的情况。

```
Philosopher Thread[Thread-4,5,main] has thought 10 times
Philosopher Thread[Thread-1,5,main] has thought 10 times
Philosopher Thread[Thread-3,5,main] has thought 10 times
Philosopher Thread[Thread-0,5,main] has thought 10 times
Philosopher Thread[Thread-2,5,main] has thought 10 times
.....
Philosopher Thread[Thread-0,5,main] has thought 690 times
Philosopher Thread[Thread-4,5,main] has thought 680 times
Philosopher Thread[Thread-1,5,main] has thought 710 times
Philosopher Thread[Thread-3,5,main] has thought 710 times
Philosopher Thread[Thread-2,5,main] has thought 710 times
Philosopher Thread[Thread-0,5,main] has thought 700 times
Philosopher Thread[Thread-4,5,main] has thought 690 times
Philosopher Thread[Thread-1,5,main] has thought 720 times
Philosopher Thread[Thread-2,5,main] has thought 720 times
Philosopher Thread[Thread-3,5,main] has thought 720 times

```

### 线程状态

执行下面命令，我们可以看一下在死锁的时候发生了什么。通过jstack我们可以到`Found 1 deadlock`，这时候可以看到，五个哲学家的线程，都locked了一个Chopstick，同时在waiting 另一个Chopstick。这也验证前面我们的分析。

```
jps | grep DiningPhilosophers| cut -d ' ' -f 1 | xargs jstack -l

结果：
....
Java stack information for the threads listed above:
===================================================
"Thread-4":
        at Philosopher.run(Philosopher.java:25)
        - waiting to lock <0x00000000d5ede300> (a Chopstick)
        - locked <0x00000000d5ede340> (a Chopstick)
"Thread-0":
        at Philosopher.run(Philosopher.java:25)
        - waiting to lock <0x00000000d5ede310> (a Chopstick)
        - locked <0x00000000d5ede300> (a Chopstick)
"Thread-1":
        at Philosopher.run(Philosopher.java:25)
        - waiting to lock <0x00000000d5ede320> (a Chopstick)
        - locked <0x00000000d5ede310> (a Chopstick)
"Thread-2":
        at Philosopher.run(Philosopher.java:25)
        - waiting to lock <0x00000000d5ede330> (a Chopstick)
        - locked <0x00000000d5ede320> (a Chopstick)
"Thread-3":
        at Philosopher.run(Philosopher.java:25)
        - waiting to lock <0x00000000d5ede340> (a Chopstick)
        - locked <0x00000000d5ede330> (a Chopstick)

Found 1 deadlock.
```

## 0x03 版本2：内置锁+全局顺序

在上一个哲学家进餐问题的实现版本中，产生了多线程编程中的经典问题：死锁。我们分析了出现死锁的原因，那么该如何解决它呢？

一个线程使用多把锁的时候，就需要考虑死锁的可能。幸运的是，有一个简单的规则可以避开死锁——总是按照一个全局的固定顺序获取多把锁。

### 改进

好，我们更新一下我们的实现程序。

**这次我们不再按照左手边和右手边的顺序拿起筷子，而是按照筷子的编号获取编号1和编号2的锁（我们并不关心编号的具体规则，只要保证编号的全局唯一且有序）。**

还是上图说比较清晰。processon里面不太适合画这种图，不过不想开visio了，凑合画了一个比较丑的。

看这个图，我们的哲学家在取筷子的时候，就不再按照之前的方式，先取左手的再取右手的筷子。我们定一个全局的筷子编号：0到4，哲学家在取筷子的时候取自己身边编号最好的筷子。



![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/philosopher.png)


假设，现在所有的所有哲学家要一起开始吃饭了，他们一起拿筷子，看哲学家0和1，他们会同时来拿0号筷子，这时候只有一个人能拿到，比如说哲学家0拿到了0号筷子，那么哲学家1就只能先等一下，只有哲学家0吃完把0号筷子放下后它才会拿，这样就不会出现死锁了。

### 代码清单 Philosopher类

下面看一下我们队哲学家类的改造。其它的类不用变。

```java
class Philosopher extends Thread {
  private Chopstick first, second;
  private Random random;
  private int thinkCount;

  public Philosopher(Chopstick left, Chopstick right) {
    if(left.getId() < right.getId()) {
      first = left; second = right;
    } else {
      first = right; second = left;
    }
    random = new Random();
  }

  public void run() {
    try {
      while(true) {
        ++thinkCount;
        if (thinkCount % 10 == 0)
          System.out.println("Philosopher " + this + " has thought " + thinkCount + " times");
        Thread.sleep(random.nextInt(10));     // Think for a while
        synchronized(first) {                   // Grab first chopstick
          synchronized(second) {                // Grab second chopstick
            Thread.sleep(random.nextInt(10)); // Eat for a while
          }
        }
      }
    } catch(InterruptedException e) {}
  }
}
```

### 运行结果

可以运行看看，这下不会出现死锁了。

```
Philosopher Thread[Thread-1,5,main] has thought 24390 times
Philosopher Thread[Thread-4,5,main] has thought 22650 times
Philosopher Thread[Thread-2,5,main] has thought 25710 times
Philosopher Thread[Thread-0,5,main] has thought 21570 times
Philosopher Thread[Thread-3,5,main] has thought 28430 times
Philosopher Thread[Thread-2,5,main] has thought 25720 times
Philosopher Thread[Thread-1,5,main] has thought 24400 times
Philosopher Thread[Thread-4,5,main] has thought 22660 times
Philosopher Thread[Thread-0,5,main] has thought 21580 times
Philosopher Thread[Thread-3,5,main] has thought 28440 times
Philosopher Thread[Thread-1,5,main] has thought 24410 times
```

## 0x04 版本3：ReentrantLock+超时取消

前面我们实现了两个版本的哲学家进餐问题，我们主要使用了Java的内置锁。但是内置锁在很多场景下是有一些缺陷的，比如我们就没有一种比较优雅的方式来终止死锁的线程。 下面我们尝试一种更优雅的锁的方式——ReentrantLock。

### 改进

这次的版本我们使用ReentrantLock来代替前两个版本的内置锁，我们用到ReentrantLock的一个特性：可以为获取锁的操作设置超时时间。

整体思路是这样的：
1. 先获取左边的筷子
2. 接着尝试获取右边的筷子，如果在指定时间内获取到了右边的筷子，就跳到步骤4，否则跳到步骤3
3. 放下左手的筷子
4. 吃饭，随后放下双手的筷子

### 代码清单 Philosopher类

```java
class Philosopher extends Thread {
  private ReentrantLock leftChopstick, rightChopstick;
  private Random random;
  private int thinkCount;

  public Philosopher(ReentrantLock leftChopstick, ReentrantLock rightChopstick) {
    this.leftChopstick = leftChopstick; this.rightChopstick = rightChopstick;
    random = new Random();
  }

  public void run() {
    try {
      while(true) {
        ++thinkCount;
        if (thinkCount % 10 == 0)
          System.out.println("Philosopher " + this + " has thought " + thinkCount + " times");
        Thread.sleep(random.nextInt(1000)); // Think for a while
        leftChopstick.lock();
        try {
          if (rightChopstick.tryLock(1000, TimeUnit.MILLISECONDS)) {
            // Got the right chopstick
            try {
              Thread.sleep(random.nextInt(1000)); // Eat for a while
            } finally { rightChopstick.unlock(); }
          } else {
            // Didn't get the right chopstick - give up and go back to thinking
            System.out.println("Philosopher " + this + " timed out");
          }
        } finally { leftChopstick.unlock(); }
      }
    } catch(InterruptedException e) {}
  }
}
```

### 运行结果

可以看到，这次基本也不会出现死锁了。

```java
Philosopher Thread[Thread-4,5,main] timed out
Philosopher Thread[Thread-2,5,main] has thought 21260 times
Philosopher Thread[Thread-2,5,main] timed out
Philosopher Thread[Thread-4,5,main] has thought 21260 times
Philosopher Thread[Thread-1,5,main] has thought 21240 times
Philosopher Thread[Thread-3,5,main] has thought 21210 times
Philosopher Thread[Thread-0,5,main] has thought 21270 times
Philosopher Thread[Thread-2,5,main] has thought 21270 times
```

### **问题分析**

我们认为这次的实现方案应该也避免了死锁，但是，真的是这样吗？

**我们使用tryLock()其实并不能避免真的避免了死锁，它只是提供了从死锁中恢复的手段。虽说死锁并没有永远的持续下去，但是对资源的争夺却没有得到任何的改善。**

而且这个方案受活锁现象的影响——如果所有的死锁线程同时超时，它们极有可能再次陷入死锁。

## 0x05 版本4：ReentrantLock+条件变量

### **条件变量**

并发编程经常需要等待某个事件的发生，比如从队列删除元素前需要等待队列非空、向缓存添加数据前需要等待缓存有足够的空间。条件变量就是为了这种情况而设计的。

建议的条件变量使用模式：

```java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();

lock.lock();
try {
    while(!<条件为真>)
        condition.await();
    <使用贡献资源>
} finally {
    lock.unlock();
}

```

一个条件变量需要与一把锁关联，线程在开始等待条件之前必须获取这把锁。获取锁后，线程检查所等待的条件是否为真，如果为真，线程将继续执行并解锁。如果条件不为真，线程将调用await方法，它将原子地解锁并阻塞等待该条件。

当另一个线程调用了signal()或者signalAll()，以为着对应的条件可能变为真，await()将原子地恢复运行并重新加锁。

### 改进

根据前面原子变量的说明，我们有了哲学家进餐问题的新解法！！！

我们对代码进行了一些调整。取掉了Chopstick类，并对Philosopher做了较大的改动。

在Philosopher类中，我们使用了条件变量，在进餐前需要判断左右邻桌是否在进餐，只有左右邻桌都没有用餐的时候他才可以进餐。也就是说，一个饥饿的哲学家一直在等这个条件：`！(left.eating || right.eating)`。



### 代码清单 Philosopher类

```java
class Philosopher extends Thread {

  private boolean eating;
  private Philosopher left;
  private Philosopher right;
  private ReentrantLock table;
  private Condition condition;
  private Random random;
  private int thinkCount;

  public Philosopher(ReentrantLock table) {
    eating = false;
    this.table = table;
    condition = table.newCondition();
    random = new Random();
  }

  public void setLeft(Philosopher left) { this.left = left; }
  public void setRight(Philosopher right) { this.right = right; }

  public void run() {
    try {
      while (true) {
        think();
        eat();
      }
    } catch (InterruptedException e) {}
  }

  private void think() throws InterruptedException {
    table.lock();
    try {
      eating = false;
      left.condition.signal();
      right.condition.signal();
    } finally { table.unlock(); }
    ++thinkCount;
    if (thinkCount % 10 == 0)
      System.out.println("Philosopher " + this + " has thought " + thinkCount + " times");
    Thread.sleep(1000);
  }

  private void eat() throws InterruptedException {
    table.lock();
    try {
      while (left.eating || right.eating)
        condition.await();
      eating = true;
    } finally { table.unlock(); }
    Thread.sleep(1000);
  }
}

```

### 结果分析

这里不再贴运行结果了。

看起来代码是比之前的复杂很多，但是性能会有显著的提升。在上一个版本中，经常会出现只有一个哲学家在进餐，其他在都持有一根筷子并在等另外一根。 在这个版本中，只要一个哲学家在理论上可进餐，那么他肯定就可以进餐。

## 0x06 总结

多线程里面的水还是很深的，一直来讲，感觉自己对它的理解都不是很深入，应该说是很浅。这次趁着清明节在家，画上一两天的时间看看书，敲敲代码还是很开心的。

文中写了四个程序，相对来讲，它们还算是循循渐进，每一个程序都会解决上一个程序的一些问题，最后逐步将多线程编程引向一个更优雅的路上。

ps：多线程这些年相对来说发展的还是挺成熟了，因此大部分内容都是自己学习后转述出来。限于个人水平和眼界的限制，暂时只能折腾到这个成都，后续继续努力改进。文中难免有很多自己疏忽和理解错误的地方，欢迎指正和交流。


## 0xFF 参考

- github地址：https://github.com/dantezhao/concurrency_and_parallelism/tree/master/dining_philosophers
- 《java并发编程实战》
- 《七天七并发模型》



***
2017-04-04 15:55 wxxy
