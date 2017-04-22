---
layout: post
title:  "Flume NG 编程实践"
categories: 漫步云端
tags: Flume
---

* content
{:toc}


## 前言

Flume已经自带了几个比较常用的source，但是在特定情况下还是有一些需求不能满足，因此需要特定开发的程序。

我们在使用的过程中，遇到了遇到对source和sink开发的情况，因此下面以这两个为例解释一下。

我们的需求主要是功能方面的，因此只写了source和sink的程序，没有对channal端没有做开发，直接用了file channal，之前看过美团对flume的使用，感觉对channal的定制开发还是不错的，感兴趣的可以参考一下。





## 定制Source


### Simple Source Example

下面是一个PollableSource的简单例子，从官网copy下来的。

主要就是configure、start、stop和process，里面的注释还是挺清晰的。

```
public class MySource extends AbstractSource implements Configurable, PollableSource {
  private String myProp;

  @Override
  public void configure(Context context) {
    String myProp = context.getString("myProp", "defaultValue");

    // Process the myProp value (e.g. validation, convert to another type, ...)

    // Store myProp for later retrieval by process() method
    this.myProp = myProp;
  }

  @Override
  public void start() {
    // Initialize the connection to the external client
  }

  @Override
  public void stop () {
    // Disconnect from external client and do any additional cleanup
    // (e.g. releasing resources or nulling-out field values) ..
  }

  @Override
  public Status process() throws EventDeliveryException {
    Status status = null;

    try {
      // This try clause includes whatever Channel/Event operations you want to do

      // Receive new data
      Event e = getSomeData();

      // Store the Event into this Source's associated Channel(s)
      getChannelProcessor().processEvent(e);

      status = Status.READY;
    } catch (Throwable t) {
      // Log exception, handle individual exceptions as needed

      status = Status.BACKOFF;

      // re-throw all Errors
      if (t instanceof Error) {
        throw (Error)t;
      }
    } finally {
      txn.close();
    }
    return status;
  }
}
```

### 定制Kafka Source

官网提供的Kafka是单topic消费的，我们在设置数据流处理系统的时候，需要对多个topic进行消费。

大致原理是这样的：

- 被消费的topic信息存放在元数据系统中；
- kafka source 初始化topic名称后消费所有的topic并传给channal；
- kafka source 端有一个线程，定时地去读取元数据系统的topic信息，这样的话，当新添加topic后，在一定的时间延迟内，flume就会继续消费新的topic。


```
public class MultiKafkaSource extends AbstractPollableSource implements Configurable {


    //初始化一些配置信息，包括元数据系统的地址rest接口。
    @Override
    public void doConfigure(Context context) {
        ......
    }

    /**
     * start sub
     */
    private void startSubscribe() {
        logger.info("startSubscribe");
        initTopics();
        if (consumer == null) {
            consumer = new KafkaConsumer<>(parameters);
            consumer.subscribe(topics);
        } else {
            consumer.unsubscribe();
            consumer.subscribe(topics);
        }

    }

    /**
     * init the topics
     */
    private void initTopics() {
        if (topics == null) {
            topics = new ArrayList<>();
        } else {
            topics.clear();
        }
        try {
            List<Message> msgList = getter.getAllMsgBasicInfo();

            for (Message message : msgList) {
                topics.add(TopicUtils.getTopic(message.getMsgType()));
            }
        } catch (IOException e) {
            logger.error("initTopics error");
            logger.error(e.getMessage());
        }

        logger.info("topics:" + Arrays.toString(topics.toArray()));
    }

    /**
     * 重新订阅，元数据系统会有更新
     */
    class ReSubscribe implements Runnable {

        @Override
        public void run() {
            while (!isStop) {
                try {
                    TimeUnit.SECONDS.sleep(subscribeTimes);
                    if (!reSubscribe.getAndSet(true)) {
                        logger.info("restart subscribe");
                    }
                } catch (Exception e) {
                    logger.error(e.getMessage());
                }
            }
        }
    }

    //开启订阅的线程
    @Override
    protected void doStart() throws FlumeException {
        logger.info("KafkaSource DO START");
        counter.start();
        startSubscribe();
        new Thread(new ReSubscribe()).start();
    }

    @Override
    protected void doStop() throws FlumeException {
        isStop = true;
        consumer.close();
        counter.stop();

        logger.info("KafkaSource Do Stop");
    }
    @Override
    protected synchronized Status doProcess() throws EventDeliveryException {
        logger.info("KafkaSource doProcess");

        long startTime = System.currentTimeMillis();
        //拉数据
        ConsumerRecords<String, String> records = consumer.poll(1000);
        //判断集群是否shutdown
        HDFSSystemState.getInstance().checkHDFSIsShutDown("source[" + flumeSourceId + "]", null);

        //检查source和sink数据量是否正常
        boolean isGood = SystemMonitor.getInstance().checkSystemIsGood(sourceProcessCounter);

        while (!isGood) {
            try {
                logger.info("source[" + flumeSourceId + "] is to fast, wait 1 minutes...");
                TimeUnit.MINUTES.sleep(1);
            } catch (InterruptedException e) {
                logger.error(e.getMessage());
            }
            isGood = SystemMonitor.getInstance().checkSystemIsGood(sourceProcessCounter);
        }

        Status status;
        if (records.isEmpty()) {
            //counter.incrementKafkaEmptyCount();
            status = Status.BACKOFF;
        } else {
            Iterator<ConsumerRecord<String, String>> iter = records.iterator();
            boolean has = true;
            while (has) {
                for (int i = 0; (has = iter.hasNext()) && i < batchLimit; i++) {
                    ConsumerRecord<String, String> record = iter.next();

                    Map<String, String> headers = new HashMap<>();
                    headers.put(EventHeader.TIMESTAME.name(),
                        String.valueOf(System.currentTimeMillis()));
                    headers.put(EventHeader.TOPIC.name(), record.topic());
                    headers.put(EventHeader.FLUME_SOURCE_ID.name(), flumeSourceId);

                    String key = record.key();
                    if (key != null && key.length() != 0) {
                        headers.put(EventHeader.KEY.name(), key);
                    }
                    //数据内容在Event中
                    Event event = EventBuilder.withBody(record.value().getBytes(), headers);
                    eventList.add(event);
                    sourceProcessCounter++;

                }

                counter.addToKafkaEventGetTimer(System.currentTimeMillis() - startTime);
                counter.addToEventReceivedCount(eventList.size());

                logger.info("KafkaSource process record size : " + eventList.size());
                if (eventList.size() > 0) {
                    getChannelProcessor().processEventBatch(eventList);

                    counter.addToEventAcceptedCount(eventList.size());

                    //buger出现的地方
                    //消费完后，kafka会记录offset。
                    //如果offset没有及时更新，下次再次启动的时候，会取到老的offset，因此就会出现重复消费数据的情况
                    consumer.commitSync();
                    eventList.clear();

                    long commitStart = System.currentTimeMillis();

                    counter.addToKafkaCommitTimer(System.currentTimeMillis() - commitStart);
                }
            }

            status = Status.READY;
        }

        if (reSubscribe.get()) {
            startSubscribe();
            reSubscribe.set(false);
        }
        return status;
    }
}
```


## 定制Sink

> The purpose of a Sink to extract Events from the Channel and forward them to the next Flume Agent in the flow or store them in an external repository. A Sink is associated with exactly one Channels, as configured in the Flume properties file. There’s one SinkRunner instance associated with every configured Sink, and when the Flume framework calls SinkRunner.start(), a new thread is created to drive the Sink (using SinkRunner.PollingRunner as the thread’s Runnable). This thread manages the Sink’s lifecycle. The Sink needs to implement the start() and stop() methods that are part of the LifecycleAware interface. The Sink.start() method should initialize the Sink and bring it to a state where it can forward the Events to its next destination. The Sink.process() method should do the core processing of extracting the Event from the Channel and forwarding it. The Sink.stop() method should do the necessary cleanup (e.g. releasing resources). The Sink implementation also needs to implement the Configurable interface for processing its own configuration settings. For example:


### Simple Example

```
public class MySink extends AbstractSink implements Configurable {
  private String myProp;

  @Override
  public void configure(Context context) {
    String myProp = context.getString("myProp", "defaultValue");

    // Process the myProp value (e.g. validation)

    // Store myProp for later retrieval by process() method
    this.myProp = myProp;
  }

  @Override
  public void start() {
    // Initialize the connection to the external repository (e.g. HDFS) that
    // this Sink will forward Events to ..
  }

  @Override
  public void stop () {
    // Disconnect from the external respository and do any
    // additional cleanup (e.g. releasing resources or nulling-out
    // field values) ..
  }

  @Override
  public Status process() throws EventDeliveryException {
    Status status = null;

    // Start transaction
    Channel ch = getChannel();
    Transaction txn = ch.getTransaction();
    txn.begin();
    try {
      // This try clause includes whatever Channel operations you want to do

      Event event = ch.take();

      // Send the Event to the external repository.
      // storeSomeData(e);

      txn.commit();
      status = Status.READY;
    } catch (Throwable t) {
      txn.rollback();

      // Log exception, handle individual exceptions as needed

      status = Status.BACKOFF;

      // re-throw all Errors
      if (t instanceof Error) {
        throw (Error)t;
      }
    }
    return status;
  }
}
```

### HDFS Sink

TBD

## 参考

- https://flume.apache.org/FlumeDeveloperGuide.html
- http://www.jianshu.com/p/befa9c06baad

***

2016-09-14 22:23:11 rljp

***

转载请注明： 转载自 [**赵德栋的博客**](http://zhaodedong.com)

作者：赵德栋，[作者介绍](http://zhaodedong.com/about/)

本博客的文章集合：http://zhaodedong.com/category/
