---
layout: post
title:  "Gradle打jar包，包含所有依赖"
categories: 代码之熵
tags: Gradle Java
---

* content
{:toc}

##  前言

最近被gradle折腾的欲仙欲死。

gradle想把所有依赖打进jar包主要有两种方式：一种是重写jar动作，一种是用第三方插件。

为了装x，我一直都是用的第一种方式，结果出了问题解决不了，为了不影响进度，只能先用第三方了。




## 重写jar动作

主要是into这一句，可以参照gradle的api文档。里面专门讲了这一块。

这种方式生成的jar包，是把所有的依赖全部打进了lib中。我一直在用这种方式打包。

然后在用spark-submit提交任务的时候，仍然会出现缺包的问题，比如我在运行spark-streaming程序的时候，就是死活找不到KafkaUtil$。但是相关的jar包的确是打进去了，百思不得其解。

所以就用了下面这种方式。



```
jar {
    manifest {  //incubating版本，以后版本可能会改API
        attributes("Main-Class": "com.KafkaWordCount",
                   "Implementation-Title": "Gradle")
    }
    into('lib') {
        from configurations.runtime
    }
}
```

## shadow

贴了一整个我的测试程序。

这种方式在使用spark提交任务的时候就OK了。

```
plugins {
    id 'com.github.johnrengelman.shadow' version '1.2.3'
    id 'java'
    id 'scala'
    id 'idea'
    id 'application'
}

group 'com.lagou'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.7
targetCompatibility = 1.7

mainClassName="com.KafkaWordCount"

repositories {
    mavenLocal()  //使用maven本地库
    mavenCentral()
}


ext{
    scala_version = '2.10.6'
    kafka_version = '0.8.2.2'
    spark_version = '1.6.0'
}

//使用shadow插件打包，代替jar功能。
jar {
    manifest {  //incubating版本，以后版本可能会改API
        attributes("Main-Class": "com.KafkaWordCount",
                   "Implementation-Title": "Gradle")
    }
}

dependencies {
    //scala
    compile "org.scala-lang:scala-library:$scala_version"
    compile "org.scala-lang:scala-compiler:$scala_version"
    //spark
    compile "org.apache.spark:spark-core_2.10:$spark_version"
    compile "org.apache.spark:spark-streaming_2.10:$spark_version"
    compile "org.apache.spark:spark-streaming-kafka_2.10:$spark_version"
    //kafka
    compile "org.apache.kafka:kafka_2.10:$kafka_version"
    compile "org.apache.kafka:kafka-clients:$kafka_version"
    //junit
    testCompile group: 'junit', name: 'junit', version: '4.11'
}
```

******
2016-03-12 19:08:00 hzct
******


来源：
http://blog.csdn.net/zhaodedong
http://zhaodedong.leanote.com
http://zhaodedong.com
