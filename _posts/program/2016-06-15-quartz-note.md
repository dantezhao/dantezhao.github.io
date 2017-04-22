---
layout: post
title:  "quartz学习笔记"
categories: 代码之熵
tags: Quartz Java
---

* content
{:toc}

## 前言

需要维护一下别人的项目，里面用到了很多组件，正好也有quartz，自己先学一下，用一下基本的api。

## 概念

Quartz核心的概念：scheduler任务调度、Job任务、Trigger触发器、JobDetail任务细节。

- Job任务：其实Job是接口，其中只有一个execute方法。开发者只要实现此接口，实现execute方法即可。把我们想做的事情，在execute中执行即可。
- JobDetail：任务细节，Quartz执行Job时，需要新建个Job实例，但是不能直接操作Job类，所以通过JobDetail来获取Job的名称、描述信息。
- Trigger触发器：执行任务的规则；比如每天，每小时等。一般情况使用SimpleTrigger，和CronTrigger，这个触发器实现了Trigger接口。
- scheduler任务调度：是最核心的概念，需要把JobDetail和Trigger注册到scheduler中，才可以执行。

## 示例

### maven依赖

```
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>
```

### cron定时的例子

自定义Job需要实现quartz的Job接口，然后在execute方法中编写需要完成的任务即可。

```
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

/**
 * Created by Dante on 2016/6/15.
 */
public class HelloJob implements Job {
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println("Hello Quartz!");
    }
}
```

编写完Job后，就用到了前面说的另外几个内容，先获取JobDetail，再创建Trigger，最后放入Scheduler中执行。

```
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

/**
 * Created by Dante on 2016/6/15.
 */
public class CronTriggerExample {

    public static void main( String[] args ) throws Exception {
        JobDetail job = JobBuilder.newJob(HelloJob.class).withIdentity(new JobKey("hello-world")).build();

        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity(new TriggerKey("hello-world")).withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")).build();

        Scheduler scheduler = new StdSchedulerFactory().getScheduler();
        scheduler.start();
        scheduler.scheduleJob(job, trigger);

    }
}
```

这样就完成了一个简单的quartz例子。

## 总结

最基本的入门例子写起来还是不难的，看看官网的文档，去google搜一两下就行了。找机会需要看一下调度系统是怎么做起来的。


***
2016-06-15 hzct
