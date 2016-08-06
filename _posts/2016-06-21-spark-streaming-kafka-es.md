---
layout: post
title:  "Spark、Spark streaming、Kafka和ES整合"
categories: 数据平台
tags: spark streaming kafka es
---

* content
{:toc}

## 前言

业务需求，记录一下实现。





## 代码

### spark 读写es

spark读es的数据

```
import org.apache.log4j.Logger
import org.apache.spark.{SparkConf, SparkContext}
import org.elasticsearch.spark._

object GetES {
  private[this] val LOG = Logger.getLogger(getClass().getName());
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("test-es").setMaster("spark://sparkip:7077")
      .set("es.index.auto.create", "false")
      .set("es.nodes", "esip")
    val sc = new SparkContext(conf)

    val data = sc.esRDD("seventest/test").collect()

    for (tmp <- data) {
      LOG.info(tmp)
    }

  }
}
```

spark写es的数据

```
import java.util.Date

import org.apache.spark.{SparkConf, SparkContext}
import org.elasticsearch.spark._

object SaveES {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("test-es").setMaster("spark://sparkip:7077")
      .set("es.index.auto.create", "false")
      .set("es.nodes", "esip")
    val sc = new SparkContext(conf)

    var se = collection.mutable.Seq[Map[String, Any]]()

    val map = Map("name" -> "zhangsan", "age" -> "20", "time" -> new Date())
    se = se :+ (map)

    sc.makeRDD(se).saveToEs("seventest/test")
  }
}
```

### spark streaming读kafka，写es

初始化context的时候，需要把es的信息填入。

```
 val conf = new SparkConf()
      .setMaster(masterUrl)
      .setAppName("kindle")
      .set("es.index.auto.create", "true")
      .set("es.nodes", "es的节点")

 val ssc = new StreamingContext(conf,
      Minutes(AppConfig.getLongconfig(Constants.CONFIG_BATCH_DURATION_KEY)))
```

kafka接入

```
 val kafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder,
      StringDecoder](ssc, KafkaUtil.getConf(), KafkaUtil.getTopics())
```

存入es。存入es的时候rdd直接调用saveToEs即可，这是es hadoop的项目里面的。

```
val esIndex = "d_test_1/test1";

  override def analytics(events: DStream[JSONObject]): this.type = {
    events.foreachRDD(rdd => {
      rdd.saveToEs(esIndex);
    })
    this
  }
```

maven依赖。

```
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch-hadoop</artifactId>
            <version>2.3.2</version>
        </dependency>
```

## 总结

没了。

***
2016-06-21 17:00:00 hzct
