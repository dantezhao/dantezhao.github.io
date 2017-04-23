
---
layout: post
title:  "漫谈并发编程：Future模型（Java、Clojure、Scala多语言角度分析）"
categories: 代码之熵
tags: Java Scala Clojure
---


## 0x00 前言


> 其实Future模型离我们并不远，如果你接触过Spark、Hadoop这些优秀的开源项目，那么在运行程序的时候关注一下他们的输出日志，一不小心你就会发现Future的身影。

在并发编程领域有很多优秀的设计模式，比如常见的Producer-Consumer模式、Pipeline模式和Future模式，这些模式都有其适用的场景，并且能够高效地解决并发问题。


这篇文章会着重分享和Future模式相关的一些知识点。




### 文章结构

本文的结构如下：

- 先解释一下什么是Future模型
- Java不可避免的是最流行的语言之一，因此我们会用Java自己实现一个Future的场景。
- 由于Java在concurrent包已经提供了对Future的支持，因此这里我们演示一下使用concurrent包的例子。
- 除了Java之外，很多语言已经在语言层面上对Future模型提供了支撑，这一部分我们用不同语言来演示Future模型。


## 0x01 Future模型简介

什么是Future模型？我们可以这样大致理解：Future模型是将异步请求和代理模式结合的产物。

为了方便理解，我们举一个场景来说明。还是假设我们是一个电商平台，用户在我们的网站下单。

如下图，用户操作的是客户端，它会向Future服务端发送数据，服务端会从后台的数据接口获取完整的订单数据，并响应用户。我们来模拟一下用户订单的行为。

1. 用户挑完商品开始下单，这时客户端向服务器端发送请求1。
2. 服务端根据客户端的信息，向后台获取完整的订单数据。这里做一个说明，比如用户客户端只发送了几个商品的id和数量，我们的服务端需要从后台数据库读取商家、商品、订单、库存等各种信息，最后拼成完整的一个订单返回。
3. 步骤2会比较耗时，因此服务端直接返回给客户端一个伪造的数据，比如一个订单id。
4. 客户端收到订单id后，开始检查订单信息，比如检查一下商品数量是否正确。**注意：** 这里如果需要付款的话，就要等到最后订单数据的返回，也就是真实的数据返回。如果数据没有返回，就要一直等待，直到返回。
5. 这时候完整的订单信息拼接完成了，返回了订单的完整数据，用户付款并完成这个订单。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/future_concept.png?x-oss-process=style/blog_dantezhao)


## 0x02 自己实现一个

这一部分我们用Java代码实现一个Future模型。

### 代码结构

如图，代码分下面几部分：

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/concurrency_and_parallelism/future_java_example.png?x-oss-process=style/blog_dantezhao)

- IData接口定义了一个数据接口，FutureData和RealData都实现了这个接口。

- FutureData是对RealData的包装，是dui真实数据的一个代理，封装了获取真实数据的等待过程。它们都实现了共同的接口，所以，针对客户端程序组是没有区别的。

- Client类向Server发送数据的请求，Sever会先返回一个Future给Client，Client收到数据后开始执行别的操作。

- 等RealData的数据完成后，会将数据返回给Client。这里的返回操作是在FutureData的getResult()。

- 因为在FutureData中的notifyAll和wait函数，主程序会等待组装完成后再会继续主进程，也就是如果没有组装完成，main函数会一直等待。

这里只做一个简单的介绍，代码中会详细解释。


注意：

    客户端在调用的方法中，单独启用一个线程来完成真实数据的组织，这对调用客户端的main函数式封闭的；
    。

### 1.代码清单 **IData接口**

FutureData和RealData都继承自IData接口。

```
/**
 * 数据的接口类
 * Created by Dante on 2017/4/8.
 */
public interface IData {
    public String getResult();
}
```

### 2.代码清单 **FutureData类**

FutureData是直接通过Server返回给客户端的数据类，这里可以理解FutureData是对真实数据RealData的一个封装。

```
/**
 * Created by Dante on 2017/4/8.
 */
/*
 * 实现了一个快速返回RealData 的包装，但并非真实的返回结果。
 */
public class FutureData  implements IData {
    protected RealData realData=null; //FutureData是RealData的一个包装
    protected boolean isReady=false;
    public synchronized void setRealData(RealData realData)
    {
        if(isReady)
        {
            return;
        }
        this.realData=realData;
        isReady=true;
        notifyAll(); //当调用Future包装类的set方法时，线程RealData被唤醒，同个getResult()方法
    }

    @Override
    public synchronized String getResult() { //会等待RealData构造完成
        while(!isReady)
        {
            try{
                wait(); //一直等待，直到RealData被注入
            }catch (Exception e)
            {}
        }
        return realData.result; //RealData的真实实现
    }
}
```

### 3.代码清单 **RealData类**

RealData是最终真实的数据，我们可以理解RealData的构造过程需要耗费十分多的时间。

```
/**
 * 真实的数据类，这是返回给用户的数据，数据的生成十分慢。
 * Created by Dante on 2017/4/8.
 */
public class RealData  implements IData {
    protected  final String result;
    public RealData(String para)
    {
        StringBuffer sb=new StringBuffer();
        //模拟一个很慢的构造过程
        for(int i=0;i<100;i++)
        {
            sb.append(para);
            try {
                Thread.sleep(100);//代替一个很慢的操作过程
            } catch (Exception e) {

            }
        }
        result=sb.toString();
    }
    public String getResult()
    {
        return result;
    }
}
```

### 4.代码清单 **Server类**

Server端，负责接收来自Client的数据请求，构造数据，并返回。

由于RealData构建很慢，因此放到一个单独的线程中进行。

**注意：** 这里会先返回一个代理的Future数据，但是在Client调用getResult()的时候，就会等待，直到真实的数据构造完成。

```
/**
 * Server端，负责接收来自Client的数据请求，构造数据，并返回
 * Created by Dante on 2017/4/8.
 */
public class Server {
    public IData request(final String queryString) {
        final FutureData future = new FutureData();
        new Thread() //RealData构建很慢，放到一个单独的线程中进行
        {
            public void run() {
                RealData realData = new RealData(queryString);
                future.setRealData(realData);
                //调用future的set方法，直接return 并唤醒RealData线程进行数据构造};
            }
        }.start();
        return future;  //立即被返回
    }
}
```

### 5.代码清单 **Client类**

代码里面注释比较详细，可以看注释理解。

```
/**
 * 主要负责调用server发起请求，并使用返回的数据
 * Created by Dante on 2017/4/8.
 */
public class Client {
    public static void main(String[] args) {
        System.out.println("建立和Server的连接！");
        Server server =new Server();
        //这里会立即返回结果，因为得到的是FutureData 而非RealData
        System.out.println("向Server发送数据请求！");
        IData data=server.request("name");
        System.out.println("请求完毕：" + data.toString() );
        try{
            //代表对其他业务的处理
            //在处理过程中，RealData被传剑，充分利用了等待时间
            //Thread.sleep(2000);
            System.out.println("开始处理其它业务！");
        }catch(Exception e)
        {}
        System.out.println("接收到真实的数据：\n"+data.getResult());
    }
}
```




## 0x03 Java concurrent包中的Future

concurrent包中的Future用起来比较方便，这里就不再做介绍，感兴趣的同学运行一下代码看看结果就清楚了。

```
/**
 * Created by Dante on 2017/4/8.
 */
import java.util.concurrent.*;

/**
 * 试验 Java 的 Future 用法
 */
public class FutureTest {

    public static class Task1 implements Callable<String> {
        @Override
        public String call() throws Exception {
            String tid = String.valueOf(Thread.currentThread().getId());
            System.out.printf("Thread#%s : in call\n", tid);
            Thread.sleep(111);
            return tid;
        }
    }

    public static class Task2 implements Callable<String> {
        @Override
        public String call() throws Exception {
            String tid = String.valueOf(Thread.currentThread().getId());
            System.out.printf("Thread#%s : in call\n", tid);
            Thread.sleep(1111);
            return tid;
        }
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService es = Executors.newCachedThreadPool();

        Future<String> restult1 = es.submit(new Task1());
        Future<String> restult2 = es.submit(new Task2());

        System.out.println(restult1.get());
        System.out.println(restult2.get());
    }

}
```

## 0x04  Scala中的Future

在scala中，Future有两种使用方式：

- 阻塞方式（Blocking）：该方式下，父actor或主程序停止执行知道所有future完成各自任务。通过scala.concurrent.Await使用。
- 非阻塞方式（Non-Blocking），也称为回调方式（Callback）：父actor或主程序在执行期间启动future，future任务和父actor并行执行，当每个future完成任务，将通知父actor。通过onComplete、onSuccess、onFailure方式使用。

*Scala这一段参考JasonDing的文章。*

### 一、阻塞方式

第一个例子展示如何创建一个future，然后通过阻塞方式等待其计算结果。虽然阻塞方式不是一个很好的用法，但是可以说明问题。

这个例子中，通过在未来某个时间计算1+1，当计算结果后再返回。


- `ExecutionContext.Implicits.global`是使用当前的全局上下文作为隐式上下文。
- `.duration._`允许我们使用`1 second`, `200 milli`样的时间间隔字面值。
- `Await.result`使用阻塞的方式等待Future任务完成, 若Future超时未完成则抛出TimeoutException异常。

```
/**
  * Created by Dante on 2017/4/23.
  */
import scala.concurrent.{Await, Future}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global

object FutureTest{

  def main(args: Array[String]) {
    val f = Future {
      println("Working on future task!")
      Thread.sleep(1000)
      1+1
    }

    println("Waiting for future task complete!")
    // 如果Future没有在Await规定的时间里返回,
    // 将抛出java.util.concurrent.TimeoutException
    val result = Await.result(f, 1 second)
    println("The future task result is " + result)
  }
}
```

### 二、非阻塞方式（回调方式）

有时你只需要监听Future的完成事件，对其进行响应，不是创建新的Future，而仅仅是产生副作用。
通过onComplete,onSuccess,onFailure三个回调函数来异步执行Future任务，而后两者仅仅是第一项的特例。

```
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Failure, Random, Success}
/**
  * Created by Dante on 2017/4/23.
  */
object NonBlockingFutureTest {
  def main(args: Array[String]) {

    println("starting calculation ...")
    val f = Future {
      Thread.sleep(Random.nextInt(500))
      42
    }

    println("before onComplete")
    f.onComplete{
      case Success(value) => println(s"Got the callback, meaning = $value")
      case Failure(e) => e.printStackTrace
    }
    // do the rest of your work
    println("A ...")
    Thread.sleep(100)
    println("B ....")
    Thread.sleep(100)
    println("C ....")
    Thread.sleep(100)
    println("D ....")
    Thread.sleep(100)
    println("E ....")
    Thread.sleep(100)
    Thread.sleep(2000)
  }
}
```

## 0x05 Clojure中的Future

Clojure是门挺有意思的语言，语法看起来比Scala恶心多了，不过适应后还是感觉挺不错的，而且通过Clojure更容易理解函数式编程。

由于Clojure用的不是很深，只是好玩学过一点，Future模型用的就更少了，为了做一个横向的对比，这里仅放一个小例子，供学习。

- Clojure在语法层面上直接支持future，使用future关键字即可。
- 使用deref或者@可以对future对象进行解引用。

```Clojure
;; Clojure在语法层面上直接支持future，使用future关键字即可
user=> (def f (future (Thread/sleep 10000) (println "done") 100))
#'user/f
;;if you wait 10 seconds before dereferencing it you'll see "done"
;; When you dereference it you will block until the result is available.
user=> @f
done
100
```

## 0xFF 总结

自己对于并发其实也是半吊子的水平，写博客主要就是一个学习的过程，这篇博客前前后后花了三个周末的时间才整完。虽说过程有点痛苦，不过收获还是挺大的。

文中不免借鉴（抄袭）了很多人的博客包括书里的内容，在后面全部列出来了。
在写博客写的时自己的思路，即使内容很多事拼接和整理而成，但是思路毕竟是自己的，文章的组织结构也是自己考虑了很久的，为了理解future也参考了好几个编程语言，包括lo这种十分小众的语言，只是最后没有写进来。


## 引用

- http://ifeve.com/promise-future-callback/
- http://blog.csdn.net/lmdcszh/article/details/39696357
- http://ifeve.com/promise-future-callback/
- http://blog.csdn.net/bboyfeiyu/article/details/24851847
- https://www.oschina.net/question/54100_83333
- https://windor.gitbooks.io/beginners-guide-to-scala/content/chp8-welcome-to-the-future.html
- https://www.kancloud.cn/digest/akka/119420
- 《七周七并发模型》

***
作者：[**dantezhao**](http://dantezhao.com) |[简书](http://www.jianshu.com/u/2453cf172ab4) | [CSDN](http://blog.csdn.net/zhaodedong) | [GITHUB](https://github.com/dantezhao)

个人主页：http://dantezhao.com
*文章可以转载, 但必须以超链接形式标明文章原始出处和作者信息*


***
2017-04-23 15:55 lkds
