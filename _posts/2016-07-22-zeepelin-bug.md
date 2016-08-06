---
layout: post
title:  "Zeppelin：和hive相关的一个bug，以及解决方案"
categories: 数据平台
tags: zeepelin hive
---

* content
{:toc}

## 前言

在用zeeplin的时候遇到一个小bug，能`select *`但是不能`select count(*)`，最后查了很久发现是一个bug，在最新版本（0.60）中仍然存在，但是有方法绕过它。




## 问题

在zeeplin中连接sql，执行下面的语句，会报空指针的错误。

```
%hive
select count(*) from trace.apptalk
```

错误：
```
java.lang.NullPointerException
	at org.apache.zeppelin.hive.HiveInterpreter.getConnection(HiveInterpreter.java:184)
	at org.apache.zeppelin.hive.HiveInterpreter.getStatement(HiveInterpreter.java:204)
	at org.apache.zeppelin.hive.HiveInterpreter.executeSql(HiveInterpreter.java:233)
	at org.apache.zeppelin.hive.HiveInterpreter.interpret(HiveInterpreter.java:328)
	at org.apache.zeppelin.interpreter.ClassloaderInterpreter.interpret(ClassloaderInterpreter.java:57)
	at org.apache.zeppelin.interpreter.LazyOpenInterpreter.interpret(LazyOpenInterpreter.java:93)
	at org.apache.zeppelin.interpreter.remote.RemoteInterpreterServer$InterpretJob.jobRun(RemoteInterpreterServer.java:300)
	at org.apache.zeppelin.scheduler.Job.run(Job.java:169)
	at org.apache.zeppelin.scheduler.ParallelScheduler$JobRunner.run(ParallelScheduler.java:157)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:178)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:292)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:744)

```



## 解决方法

```
%hive(default)
select count(*) from trace.talk
```


## 参考：

https://community.hortonworks.com/questions/23197/nullpointerexception-on-query.html


http://mail-archives.apache.org/mod_mbox/incubator-zeppelin-users/201602.mbox/%3CFFE89F93-5B33-469E-98D3-4C34ED351FA6@gmail.com%3E

https://issues.apache.org/jira/browse/ZEPPELIN-628?jql=project%20%3D%20ZEPPELIN

https://mail-archives.apache.org/mod_mbox/incubator-zeppelin-users/201601.mbox/%3CCAEkQ2NMyq9q7jJFBXkONJW6noJTQYii6UpKjg05Aewu32pSDiw@mail.gmail.com%3E

***
2016-07-22 13:37:25 hzct
