---
layout: post
title:  "Hive：一次错误解决的详细记录（关于内存和权限）"
categories: 漫步云端
tags: Hive
---

* content
{:toc}

## 前言

有那么一些大大的sql，是用来建各种宽表的。这些操作对impala来说，压力还是比较大的，而且在这个时候正好有别人的impala查询什么的，经常会报错。这是背景，因此我尝试用hive看能不能解决。

下面这个错误，之前解决过，但是随着数据量的增大，即使扩大了impala的内存限制，早晚还是会出问题了。




```
Memory limit exceeded Cannot perform hash aggregation. Partitioned input data too many times. This could mean there is too much skew in the data or the memory limit is set too low.
```

## 过程


### **阶段一：** 初始错误

执行的命令，是一个大大的sql。下面是错误。

```
Total jobs = 5
Launching Job 1 out of 5
Number of reduce tasks not specified. Estimated from input data size: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
java.io.IOException: org.apache.hadoop.yarn.exceptions.InvalidResourceRequestException: Invalid resource request, requested memory < 0, or requested memory > max configured, requestedMemory=16384, maxMemory=8192
	at org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerUtils.validateResourceRequest(SchedulerUtils.java:196)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.validateResourceRequest(RMAppManager.java:387)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.createAndPopulateNewRMApp(RMAppManager.java:347)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.submitApplication(RMAppManager.java:271)
	at org.apache.hadoop.yarn.server.resourcemanager.ClientRMService.submitApplication(ClientRMService.java:538)
	at org.apache.hadoop.yarn.api.impl.pb.service.ApplicationClientProtocolPBServiceImpl.submitApplication(ApplicationClientProtocolPBServiceImpl.java:188)
	at org.apache.hadoop.yarn.proto.ApplicationClientProtocol$ApplicationClientProtocolService$2.callBlockingMethod(ApplicationClientProtocol.java:323)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:587)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1026)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2013)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2009)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2007)

	at org.apache.hadoop.mapred.YARNRunner.submitJob(YARNRunner.java:305)
	at org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:528)
	at org.apache.hadoop.mapreduce.Job$10.run(Job.java:1295)
	at org.apache.hadoop.mapreduce.Job$10.run(Job.java:1292)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.mapreduce.Job.submit(Job.java:1292)
	at org.apache.hadoop.mapred.JobClient$1.run(JobClient.java:564)
	at org.apache.hadoop.mapred.JobClient$1.run(JobClient.java:559)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.mapred.JobClient.submitJobInternal(JobClient.java:559)
	at org.apache.hadoop.mapred.JobClient.submitJob(JobClient.java:550)
	at org.apache.hadoop.hive.ql.exec.mr.ExecDriver.execute(ExecDriver.java:420)
	at org.apache.hadoop.hive.ql.exec.mr.MapRedTask.execute(MapRedTask.java:136)
	at org.apache.hadoop.hive.ql.exec.Task.executeTask(Task.java:153)
	at org.apache.hadoop.hive.ql.exec.TaskRunner.runSequential(TaskRunner.java:85)
	at org.apache.hadoop.hive.ql.Driver.launchTask(Driver.java:1554)
	at org.apache.hadoop.hive.ql.Driver.execute(Driver.java:1321)
	at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1139)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:962)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:952)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:269)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:221)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:431)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:800)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:694)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:633)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:212)
Caused by: org.apache.hadoop.yarn.exceptions.InvalidResourceRequestException: Invalid resource request, requested memory < 0, or requested memory > max configured, requestedMemory=16384, maxMemory=8192
	at org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerUtils.validateResourceRequest(SchedulerUtils.java:196)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.validateResourceRequest(RMAppManager.java:387)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.createAndPopulateNewRMApp(RMAppManager.java:347)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.submitApplication(RMAppManager.java:271)
	at org.apache.hadoop.yarn.server.resourcemanager.ClientRMService.submitApplication(ClientRMService.java:538)
	at org.apache.hadoop.yarn.api.impl.pb.service.ApplicationClientProtocolPBServiceImpl.submitApplication(ApplicationClientProtocolPBServiceImpl.java:188)
	at org.apache.hadoop.yarn.proto.ApplicationClientProtocol$ApplicationClientProtocolService$2.callBlockingMethod(ApplicationClientProtocol.java:323)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:587)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1026)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2013)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2009)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2007)

	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
	at org.apache.hadoop.yarn.ipc.RPCUtil.instantiateException(RPCUtil.java:53)
	at org.apache.hadoop.yarn.ipc.RPCUtil.unwrapAndThrowException(RPCUtil.java:101)
	at org.apache.hadoop.yarn.api.impl.pb.client.ApplicationClientProtocolPBClientImpl.submitApplication(ApplicationClientProtocolPBClientImpl.java:211)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:187)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
	at com.sun.proxy.$Proxy29.submitApplication(Unknown Source)
	at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.submitApplication(YarnClientImpl.java:225)
	at org.apache.hadoop.mapred.ResourceMgrDelegate.submitApplication(ResourceMgrDelegate.java:282)
	at org.apache.hadoop.mapred.YARNRunner.submitJob(YARNRunner.java:289)
	... 34 more
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.yarn.exceptions.InvalidResourceRequestException): Invalid resource request, requested memory < 0, or requested memory > max configured, requestedMemory=16384, maxMemory=8192
	at org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerUtils.validateResourceRequest(SchedulerUtils.java:196)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.validateResourceRequest(RMAppManager.java:387)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.createAndPopulateNewRMApp(RMAppManager.java:347)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.submitApplication(RMAppManager.java:271)
	at org.apache.hadoop.yarn.server.resourcemanager.ClientRMService.submitApplication(ClientRMService.java:538)
	at org.apache.hadoop.yarn.api.impl.pb.service.ApplicationClientProtocolPBServiceImpl.submitApplication(ApplicationClientProtocolPBServiceImpl.java:188)
	at org.apache.hadoop.yarn.proto.ApplicationClientProtocol$ApplicationClientProtocolService$2.callBlockingMethod(ApplicationClientProtocol.java:323)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:587)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1026)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2013)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2009)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2007)

	at org.apache.hadoop.ipc.Client.call(Client.java:1411)
	at org.apache.hadoop.ipc.Client.call(Client.java:1364)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:206)
	at com.sun.proxy.$Proxy28.submitApplication(Unknown Source)
	at org.apache.hadoop.yarn.api.impl.pb.client.ApplicationClientProtocolPBClientImpl.submitApplication(ApplicationClientProtocolPBClientImpl.java:208)
	... 44 more
Job Submission failed with exception 'java.io.IOException(org.apache.hadoop.yarn.exceptions.InvalidResourceRequestException: Invalid resource request, requested memory < 0, or requested memory > max configured, requestedMemory=16384, maxMemory=8192
	at org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerUtils.validateResourceRequest(SchedulerUtils.java:196)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.validateResourceRequest(RMAppManager.java:387)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.createAndPopulateNewRMApp(RMAppManager.java:347)
	at org.apache.hadoop.yarn.server.resourcemanager.RMAppManager.submitApplication(RMAppManager.java:271)
	at org.apache.hadoop.yarn.server.resourcemanager.ClientRMService.submitApplication(ClientRMService.java:538)
	at org.apache.hadoop.yarn.api.impl.pb.service.ApplicationClientProtocolPBServiceImpl.submitApplication(ApplicationClientProtocolPBServiceImpl.java:188)
	at org.apache.hadoop.yarn.proto.ApplicationClientProtocol$ApplicationClientProtocolService$2.callBlockingMethod(ApplicationClientProtocol.java:323)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:587)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1026)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2013)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2009)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2007)
)'
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask
```

**解决思路：**

出现这个问题，一看就是hadoop自身的嘛，而且是和内存相关的，不巧我之前正好专门解决过类似的问题，那就搞一下呗。

我看了一下集群的配置和错误信息，嗯，先调一下参数。把下面的额参数调成16G的。
注意，只改一个还是会报错的。

```
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>8192</value>
  </property>
   <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>8192</value>
  </property>

```

### **阶段二：** Error: Java heap space

调完参数就再运行一下呗。我就想说，搞数据研发，天生就是来填坑的。


```

Total jobs = 5
Launching Job 1 out of 5
Number of reduce tasks not specified. Estimated from input data size: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1461315608587_0001, Tracking URL = http://ip:8088/proxy/
Kill Command = /usr/lib/hadoop/bin/hadoop job  -kill job_1461315608587_0001
Hadoop job information for Stage-1: number of mappers: 3; number of reducers: 1
2016-04-22 17:01:24,128 Stage-1 map = 0%,  reduce = 0%
2016-04-22 17:01:50,217 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_1461315608587_0001 with errors
Error during job, obtaining debugging information...
Examining task ID: task_1461315608587_0001_m_000000 (and more) from job job_1461315608587_0001

Task with the most failures(4):
-----
Task ID:
  task_1461315608587_0001_m_000000

URL:
  http://ip:8088/taskdetails.jsp?jobid=job_1461315608587_0001&tipid=task_14
-----
Diagnostic Messages for this Task:
Error: Java heap space

FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask
MapReduce Jobs Launched:
Stage-Stage-1: Map: 3  Reduce: 1   HDFS Read: 0 HDFS Write: 0 FAIL
Total MapReduce CPU Time Spent: 0 msec
```


下面这个问题就不开心了，相当于是内存的各个设置值是不合理的。于是我找一下任务的详细错误日志。

下面这个错误看来还是内存的问题。好，我查配置文件，自己算一下内存情况。


```
Log Type: stderr
Log Length: 0
Log Type: stdout
Log Length: 0
Log Type: syslog
Log Length: 5477
Showing 4096 bytes of 5477 total. Click here for the full log.
m started
2016-04-22 17:19:22,376 INFO [main] org.apache.hadoop.mapred.YarnChild: Executing with tokens:
2016-04-22 17:19:22,376 INFO [main] org.apache.hadoop.mapred.YarnChild: Kind: mapreduce.job, Service: job_1461315608587_0016, Ident: (org.apache.hadoop.mapreduce.security.token.JobTokenIdentifier@528ca407)
2016-04-22 17:19:22,466 INFO [main] org.apache.hadoop.mapred.YarnChild: Sleeping for 0ms before retrying again. Got null now.
2016-04-22 17:19:22,760 INFO [main] org.apache.hadoop.mapred.YarnChild: mapreduce.cluster.local.dir for child: /data/logs/nm/usercache/root/appcache/application_1461315608587_0016
2016-04-22 17:19:22,859 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: hadoop.ssl.require.client.cert;  Ignoring.
2016-04-22 17:19:22,860 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: mapreduce.job.end-notification.max.retry.interval;  Ignoring.
2016-04-22 17:19:22,861 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: hadoop.ssl.client.conf;  Ignoring.
2016-04-22 17:19:22,862 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: hadoop.ssl.keystores.factory.class;  Ignoring.
2016-04-22 17:19:22,863 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: hadoop.ssl.server.conf;  Ignoring.
2016-04-22 17:19:22,870 WARN [main] org.apache.hadoop.conf.Configuration: job.xml:an attempt to override final parameter: mapreduce.job.end-notification.max.attempts;  Ignoring.
2016-04-22 17:19:23,173 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2016-04-22 17:19:23,694 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2016-04-22 17:19:23,985 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: Paths:/lagou/data/mysql/lagou/position_detail/part-m-00000:134217728+134217728InputFormatClass: org.apache.hadoop.mapred.TextInputFormat

2016-04-22 17:19:24,074 INFO [main] org.apache.hadoop.hive.ql.log.PerfLogger: <PERFLOG method=deserializePlan from=org.apache.hadoop.hive.ql.exec.Utilities>
2016-04-22 17:19:24,075 INFO [main] org.apache.hadoop.hive.ql.exec.Utilities: Deserializing MapWork via kryo
2016-04-22 17:19:24,293 INFO [main] org.apache.hadoop.hive.ql.log.PerfLogger: </PERFLOG method=deserializePlan start=1461316764074 end=1461316764293 duration=219 from=org.apache.hadoop.hive.ql.exec.Utilities>
2016-04-22 17:19:24,324 INFO [main] org.apache.hadoop.hive.ql.io.HiveContextAwareRecordReader: Processing file hdfs://plat-main-cdh-bjc-001:8020/lagou/data/mysql/lagou/position_detail/part-m-00000
2016-04-22 17:19:24,324 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: map.input.file is deprecated. Instead, use mapreduce.map.input.file
2016-04-22 17:19:24,324 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: map.input.start is deprecated. Instead, use mapreduce.map.input.start
2016-04-22 17:19:24,324 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: map.input.length is deprecated. Instead, use mapreduce.map.input.length
2016-04-22 17:19:24,324 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 1
2016-04-22 17:19:24,534 FATAL [main] org.apache.hadoop.mapred.YarnChild: Error running child : java.lang.OutOfMemoryError: Java heap space
	at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.init(MapTask.java:983)
	at org.apache.hadoop.mapred.MapTask.createSortingCollector(MapTask.java:401)
	at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:439)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:343)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:168)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1642)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:163)

```


### **阶段三：** hive客户端配置的坑

我自己突然想起来，集群内存那么大，不应该出这个问题啊，而且按照配置来说也不应该出错，这时候我想到了一个问题，hive客户端和集群的不对的话，也会出问题。还好我之前被客户端坑过。

把下面的配置文件都同步一下就OK了，不报这些乱七八糟的错误了。

这是客户端的：

```
<property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Djava.net.preferIPv4Stack=true  -Xmx17000m -Xmx12884901888</value>
  </property>
  <property>
    <name>mapreduce.map.java.opts</name>
    <value>-Djava.net.preferIPv4Stack=true -Xmx512m -Xmx134217728</value>
  </property>
  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Djava.net.preferIPv4Stack=true -Xmx1024m -Xmx536870912</value>
  </property>
  <property>
```
这是集群的：

```
<property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Djava.net.preferIPv4Stack=true -Xmx825955249</value>
  </property>
  <property>
    <name>mapreduce.map.java.opts</name>
    <value>-Djava.net.preferIPv4Stack=true -Xmx825955249</value>
  </property>
  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Djava.net.preferIPv4Stack=true -Xmx825955249</value>
  </property>

```

### **阶段四：** 权限

当然没这么容易就好啊。

请注意下面的运行的结果中的这一句：`Failed with exception Unable to move source hdfs://ip:8020/data/hive/warehouse/.hive-staging_hive_2016-04-22_18-13-10_912_2927332462619031407-1/-ext-10001 to destination`。

这是一个权限问题。好吧，又跑到权限上来了。那我就su一下hive呗，反正是测试嘛。直接su一下。

```
MapReduce Total cumulative CPU time: 1 minutes 28 seconds 280 msec
Ended Job = job_1461315608587_0050
Moving data to: hdfs://plat-main-cdh-bjc-001:8020/data/hive/warehouse/dante_test.db/po
Failed with exception Unable to move source hdfs://ip:8020/data/hive/warehouse/.hive-staging_hive_2016-04-22_18-13-10_912_2927332462619031407-1/-ext-10001 to destination hdfs://ip:8020/data/hive/warehouse/dante_test.db/po
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.MoveTask
MapReduce Jobs Launched:
Stage-Stage-1: Map: 3  Reduce: 1   Cumulative CPU: 31.92 sec   HDFS Read: 316017240 HDFS Write: 54293815 SUCCESS
Stage-Stage-2: Map: 4  Reduce: 1   Cumulative CPU: 57.09 sec   HDFS Read: 370311430 HDFS Write: 169115178 SUCCESS
Stage-Stage-3: Map: 5  Reduce: 1   Cumulative CPU: 88.28 sec   HDFS Read: 721889477 HDFS Write: 300934764 SUCCESS
Total MapReduce CPU Time Spent: 2 minutes 57 seconds 290 msec
```

### **阶段五：** hive用户登录

看到下面的结果我也是醉了，第一次还没发现，后来才看出来原来没切换成功。

```
[root@ ~]$ su hive
[root@ ~]$
```

我看一下系统的日志,基本是登录后秒被踢出的赶脚。

```
$ tail -f  /var/log/secure
Apr 22 19:01:05 hadoop-cluster-8-228 su: pam_unix(su:session): session opened for user hive by root(uid=0)
Apr 22 19:01:05 hadoop-cluster-8-228 su: pam_unix(su:session): session closed for user hive
```
问了一下ruby，嗯，是hive用户本来就不让登录（我承认我是忘了这茬了）。下面是`/etc/passwd`中的信息。

```
hive:x:486:484:Hive:/var/lib/hive:/bin/false
```


### **阶段六：** 使用impala用户

到目前为止，我想到的解决的方法有两种：修改hdfs中目录权限，修改hive用户的登录权限。但是这两个我其实都不想改，随便改权限没什么好的。我就是不改。

这个时候我就随便关翻翻呗，看到hive下面的权限居然还有一个impala。

```
$ hadoop fs -ls /data/hive/warehouse
Found 19 items
drwxrwxrwt   - impala hive          0 2016-04-22 19:07 /data/hive/warehouse/.hive-staging_hive_2016-04-22_19-07-01_479_7625728582998415102-1
drwxrwxrwt   - impala hive          0 2016-04-22 16:16 /data/hive/warehouse/dante_test.db
```

正好impala也能登录，那就用impala用户来执行hive，然后根据错误提示给impala用户在/tmp下建两个文件夹就OK了。再执行就不会报错了。


下面就是没有错误的结果了。

```
MapReduce Total cumulative CPU time: 1 minutes 29 seconds 760 msec
Ended Job = job_1461315608587_0133
Moving data to: hdfs://ip:8020/data/hive/warehouse/dante_test.db/po
Table dante_test.po stats: [numFiles=1, numRows=0, totalSize=300934672, rawDataSize=0]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 3  Reduce: 1   Cumulative CPU: 33.0 sec   HDFS Read: 316017240 HDFS Write: 54293815 SUCCESS
Stage-Stage-2: Map: 4  Reduce: 1   Cumulative CPU: 57.65 sec   HDFS Read: 370311432 HDFS Write: 169115178 SUCCESS
Stage-Stage-3: Map: 5  Reduce: 1   Cumulative CPU: 89.76 sec   HDFS Read: 721889481 HDFS Write: 300934764 SUCCESS
Total MapReduce CPU Time Spent: 3 minutes 0 seconds 410 msec
OK
Time taken: 164.625 seconds

```

## 总结

使用hive的一次小坑的解决经过。幸好都报错错误了，不然打死了不会再整理了。

目前的hive还没有跑在脚本里面，不过明天应该就开始完整地弄一个hive的脚本，到时候就该改hive在hdfs中目录的权限了，或者是把调度上的用户添加到hive的权限组中。

目前hive还没有深入用，但是有几个坑是肯定存在的，想都不用想。最大的坑之一肯定是impala和hive的元数据同步的问题，这个问题，单纯是impala自身的都还没有玩明白，加上hive肯定会死人的。

***
2016-04-22 20:14:00 hzct
