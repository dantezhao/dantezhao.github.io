---
layout: post
title:  "Presto安装过程错误记录"
categories: 漫步云端
tags: Presto
---

* content
{:toc}

## 前言

presto安装过程中错误记录。




## 错误记录

### 1.错误：

```
1) Error: Defunct property 'task.max-memory' (class [class com.facebook.presto.execution.TaskManagerConfig]) cannot be configured.
  at com.facebook.presto.server.ServerMainModule.setup(ServerMainModule.java:254)

2) Configuration property 'task.max-memory=1GB' was not used
  at io.airlift.bootstrap.Bootstrap.lambda$initialize$2(Bootstrap.java:235)

2 errors
com.google.inject.CreationException: Unable to create injector, see the following errors:

1) Error: Defunct property 'task.max-memory' (class [class com.facebook.presto.execution.TaskManagerConfig]) cannot be configured.
  at com.facebook.presto.server.ServerMainModule.setup(ServerMainModule.java:254)

2) Configuration property 'task.max-memory=1GB' was not used
  at io.airlift.bootstrap.Bootstrap.lambda$initialize$2(Bootstrap.java:235)

2 errors
	at com.google.inject.internal.Errors.throwCreationExceptionIfErrorsExist(Errors.java:466)
	at com.google.inject.internal.InternalInjectorCreator.initializeStatically(InternalInjectorCreator.java:155)
	at com.google.inject.internal.InternalInjectorCreator.build(InternalInjectorCreator.java:107)
	at com.google.inject.Guice.createInjector(Guice.java:96)
	at io.airlift.bootstrap.Bootstrap.initialize(Bootstrap.java:242)
	at com.facebook.presto.server.PrestoServer.run(PrestoServer.java:111)
	at com.facebook.presto.server.PrestoServer.main(PrestoServer.java:63)
```

原因和解决方法：

配置文件的版本不对，按照自己版本的配置即可消除此错误。

### 2.错误：

```
2016-07-28T16:14:00.631+0800	ERROR	Announcer-0	io.airlift.discovery.client.Announcer	Service announcement failed after 557.75ms. Next request will happen within 1000.00ms
2016-07-28T16:14:01.798+0800	ERROR	Announcer-1	io.airlift.discovery.client.Announcer	Service announcement failed after 165.75ms. Next request will happen within 1000.00ms
2016-07-28T16:14:03.079+0800	ERROR	Announcer-4	io.airlift.discovery.client.Announcer	Service announcement failed after 280.31ms. Next request will happen within 1000.00ms
2016-07-28T16:14:04.638+0800	ERROR	Announcer-0	io.airlift.discovery.client.Announcer	Service announcement failed after 557.94ms. Next request will happen within 1000.00ms
2016-07-28T16:14:05.837+0800	ERROR	Announcer-1	io.airlift.discovery.client.Announcer	Service announcement failed after 197.69ms. Next request will happen within 1000.00ms
2016-07-28T16:14:07.086+0800	ERROR	Announcer-4	io.airlift.discovery.client.Announcer	Service announcement failed after 248.15ms. Next request will happen within 1000.00ms
```

原因和解决

stack overflow上的说明如下

> This error message occurs because the service selectors and announcer start trying to connect while the server is still starting. You should see "succeeded for refresh" and "succeeded for announce" shortly after in the logs which shows that it's working. We will fix the log message eventually but it's purely a cosmetic issue.

目前的确是这样的，jps一下就可以看到PrestoServer的进程在，干掉重来即可。


### 3. 错误：

```
presto:default> show schemas from hive;
Query 20160801_074338_00017_aw9xp failed: Catalog hive does not exist
```
任何查询相关的语句都会报这个错。

原因及解决

stack overflow上的哥们发的：

> I'm not sure what changed. I double checked the discovery.uri setting and it was already correct. I restarted the coordinator and worker and now I can query using the presto-cli pointing to the coordinator.

我一想，应该是修改了配置文件，但是没有重启presto的原因，重启一下即可。

***
2016-08-05 20:22:22 hzct
