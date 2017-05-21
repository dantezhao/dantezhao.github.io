
## 0x00 前言

> 本篇是Spark源码解析的第一篇，主要通过源码分析Spark设计中最重要的一个概念——RDD。

本文会主要讲解RDD的主要概念和源码中的设计，并通过一个例子详细地讲解RDD是如何生成的和转换的。


### 文章结构

1. 先回顾一下RDD的一些特征以及几个基本概念
2. RDD源码分析，整体的源码设计
3. 举一个例子，通过这个例子来一步步地追踪源码。


## 0x01 概念

### 什么是RDD

RDD（Resilient Distributed Dataset）：弹性分布式数据集。

我们可以先大致这样理解RDD：RDD是一个容错的、并行的**数据结构**，可以让用户显式地将数据存储到磁盘和内存中，并能控制数据的分区。同时，RDD还提供了一组丰富的操作来处理这些数据。

**注意**：RDD作为数据结构，本质上是一个只读的分区记录集合。一个RDD可以包含多个分区，每个分区就是一个dataset片段。RDD可以相互依赖。

### RDD的5个特征

下面是源码中对RDD类的注释：

> Internally, each RDD is characterized by five main properties:

> - A list of partitions
> - A function for computing each split
> - A list of dependencies on other RDDs
> - Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
> - Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)

也是说RDD会有5个基本特征:

1. 有一个分片列表。就是能被切分，和hadoop一样的，能够切分的数据才能并行计算。

2. 有一个函数计算每一个分片，这里指的是下面会提到的compute函数。

3. 对其他的RDD的依赖列表，依赖还具体分为宽依赖和窄依赖。

4. 可选：key-value型的RDD是根据哈希来分区的，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce。

5. 可选：每一个分片的优先计算位置（preferred locations），比如HDFS的block的所在位置应该是优先计算的位置。

### 宽窄依赖

这里有必要稍微解释一下窄依赖（narrow dependency）和宽依赖（wide dependency）。

> 如果RDD的每个分区最多只能被一个Child RDD的一个分区使用，则称之为narrow dependency；若多个Child RDD分区都可以依赖，则称之为wide dependency。不同的操作依据其特性，可能会产生不同的依赖。

例如map操作会产生narrow dependency，而join操作则产生wide dependency。


如图，两种依赖的区别：

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/source_code/spark-source-code-rdd-1.png?x-oss-process=style/blog_dantezhao)


## 0x02 源码分析

RDD的5个特征会对应到源码中的4个方法和一个属性。

`RDD.scala`是一个总的抽象，不同的子类会对下面的方法进行定制化的实现。比如compute方法，不同的子类在实现的时候是不同的。 下面会对每一块单独分析。

```
  //该方法只会被调用一次。由子类实现，返回这个RDD的所有partition。
  protected def getPartitions: Array[Partition]
  //该方法只会被调用一次。计算该RDD和父RDD的依赖关系
  protected def getDependencies: Seq[Dependency[_]] = deps
  // 对分区进行计算，返回一个可遍历的结果
  def compute(split: Partition, context: TaskContext): Iterator[T]
  //可选的，指定优先位置，输入参数是split分片，输出结果是一组优先的节点位置
  protected def getPreferredLocations(split: Partition): Seq[String] = Nil
  //可选的，分区的方法，针对第4点，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce
  @transient val partitioner: Option[Partitioner] = None
```


### 举个栗子

官网最基本的wordcount例子。虽简单，但是代表性很强。

```
val textFile = sc.textFile("hdfs://...")
val counts = textFile.flatMap(line => line.split(" "))
                 .filter(_.length >= 2)
                 .map(word => (word, 1))
                 .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```

这里涉及到了下面几个RDD转换：

1. textFile是一个HadoopRDD经过map后的MapPartitionsRDD，
2. 经过flatMap后仍然是一个MapPartitionsRDD，
3. 经过filter方法之后生成了一个新的MapPartitionsRDD，
4. 经过map函数之后，继续是一个MapPartitionsRDD，
5. 最后经过reduceByKey变成了ShuffleRDD。

在正式看源码之前，上一个图。 这个图是整个流程中RDD的转换过程，这里先不讲解，后面看源码的时候如果有疑惑再回过头来看，就明白了。

![](http://dantezhao.oss-cn-shanghai.aliyuncs.com/source_code/spark-source-code-rdd-2.png?x-oss-process=style/blog_dantezhao)

### 1. 源码分析：`SparkContext`类

我们首先看textFile的这个方法，在SparkContext中。

看注释：

> Read a text file from HDFS, a local file system (available on all nodes), or any Hadoop-supported file system URI, and return it as an RDD of Strings.

其实textFile只是对hadoopFile方法做了一层封装。

**注意：** 此处有一个比较长的关系链，为了理解textfile中的逻辑，需要先看hadoopFile；hadoopFile最后返回的是一个HadoopRDD对象，然后HadoopRDD经过map变换后，转换成MapPartitionsRDD，由于HadoopRDD没有重写map函数，因此调用的是父类RDD的map；


```
  def textFile(path: String, minPartitions: Int = defaultMinPartitions): RDD[String] = withScope {
    assertNotStopped()//暂时不用看
    hadoopFile(path, classOf[TextInputFormat], classOf[LongWritable], classOf[Text], minPartitions).map(pair => pair._2.toString).setName(path)
  }
```


我们继续向下追，看一下hadoopFile方法，hadoopFile中做了这些事。

- 把hadoop的配置文件保存到广播变量里；
- 设置路径的方法；
- new了一个HadoopRDD,并返回。

```
  def hadoopFile[K, V](path: String,inputFormatClass: Class[_ <: InputFormat[K, V]],keyClass: Class[K],valueClass: Class[V],minPartitions: Int = defaultMinPartitions): RDD[(K, V)] = withScope {
    assertNotStopped()
    // A Hadoop configuration can be about 10 KB, which is pretty big, so broadcast it.
    val confBroadcast = broadcast(new SerializableConfiguration(hadoopConfiguration))
    val setInputPathsFunc = (jobConf: JobConf) => FileInputFormat.setInputPaths(jobConf, path)
    new HadoopRDD(this, confBroadcast,Some(setInputPathsFunc),inputFormatClass,keyClass,=valueClass,minPartitions).setName(path)
  }
```

我们看一下它的输入参数。如果你写过MR程序，是不是特别熟悉？是不是和Mapper的API和接近？

```
Mapper<Object, Text, Text, IntWritable>
```

### 2. 源码分析：`HadoopRDD`类

下面专门来看一下HadoopRDD是干什么的。

> An RDD that provides core functionality for reading data stored in Hadoop (e.g., files in HDFS, sources in HBase, or S3)

看注释可以知道，HadoopRDD是一个专为Hadoop（HDFS、Hbase、S3）设计的RDD。

HadoopRDD主要重写了三个方法，可以在源码中找到加override标识的方法：

- `override def getPartitions: Array[Partition]`
- `override def compute(theSplit: Partition, context: TaskContext): InterruptibleIterator[(K, V)] `
- `override def getPreferredLocations(split: Partition): Seq[String] `

下面分别看一下这三个方法。

getPartitions方法最后是返回了一个array。它调用的是inputFormat自带的getSplits方法来计算分片，然后把分片信息放到array中。

**这里，我们是不是就可以理解，Hadoop中的一个分片，就对应到Spark中的一个Partition。**

```
  override def getPartitions: Array[Partition] = {
    val jobConf = getJobConf()
    // add the credentials here as this can be called before SparkContext initialized
    SparkHadoopUtil.get.addCredentials(jobConf)
    val inputFormat = getInputFormat(jobConf)
    val inputSplits = inputFormat.getSplits(jobConf, minPartitions)
    val array = new Array[Partition](inputSplits.size)
    for (i <- 0 until inputSplits.size) {
      array(i) = new HadoopPartition(id, i, inputSplits(i))
    }
    array
  }
```

compute方法的作用主要就是根据输入的partition信息生成一个InterruptibleIterator。如下面代码段。

```
override def compute(theSplit: Partition, context: TaskContext): InterruptibleIterator[(K, V)] = {
    val iter = new NextIterator[(K, V)] {......}
    new InterruptibleIterator[(K, V)](context, iter)
  }
```

下面看一下iter中做了什么逻辑处理。

1. 把Partition转成HadoopPartition，然后通过InputSplit创建一个RecordReader
2. 重写Iterator的getNext方法，通过创建的reader调用next方法读取下一个值。

从这里我们可以看得出来compute方法是通过分片来获得Iterator接口，以遍历分片的数据。

```
override def compute(theSplit: Partition, context: TaskContext): InterruptibleIterator[(K, V)] = {

 val iter = new NextIterator[(K, V)] {

      //将compute的输入theSplit，转换为HadoopPartition
      val split = theSplit.asInstanceOf[HadoopPartition]
      ......
      //c重写getNext方法
      override def getNext(): (K, V) = {
        try {
          finished = !reader.next(key, value)
        } catch {
          case _: EOFException if ignoreCorruptFiles => finished = true
        }
        if (!finished) {
          inputMetrics.incRecordsRead(1)
        }
        (key, value)
      }
     }
}
```

getPreferredLocations方法比较简单，直接调用SplitInfoReflections下的inputSplitWithLocationInfo方法获得所在的位置。

```
  override def getPreferredLocations(split: Partition): Seq[String] = {
    val hsplit = split.asInstanceOf[HadoopPartition].inputSplit.value
    val locs: Option[Seq[String]] = HadoopRDD.SPLIT_INFO_REFLECTIONS match {
      case Some(c) =>
        try {
          val lsplit = c.inputSplitWithLocationInfo.cast(hsplit)
          val infos = c.getLocationInfo.invoke(lsplit).asInstanceOf[Array[AnyRef]]
          Some(HadoopRDD.convertSplitLocationInfo(infos))
        } catch {
          case e: Exception =>
            logDebug("Failed to use InputSplitWithLocations.", e)
            None
        }
      case None => None
    }
    locs.getOrElse(hsplit.getLocations.filter(_ != "localhost"))
  }
```

### 3. 源码分析：`MapPartitionsRDD`类

先看一在RDD类中的map方法。

> Return a new RDD by applying a function to all elements of this RDD.

它最后返回的是一个MapPartitionsRDD。并且对RDD中的每一个元素都调用了一个function。

```
  /**
   *
   */
  def map[U: ClassTag](f: T => U): RDD[U] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
  }
```

那么MapPartitionsRDD是干什么的呢。

> An RDD that applies the provided function to every partition of the parent RDD.

可以看到，它重写了父类RDD的partitioner、getPartitions和compute。

```
private[spark] class MapPartitionsRDD[U: ClassTag, T: ClassTag](
    var prev: RDD[T],
    f: (TaskContext, Int, Iterator[T]) => Iterator[U],  // (TaskContext, partition index, iterator)
    preservesPartitioning: Boolean = false)
  extends RDD[U](prev) {
  override val partitioner = if (preservesPartitioning) firstParent[T].partitioner else None
  override def getPartitions: Array[Partition] = firstParent[T].partitions
  override def compute(split: Partition, context: TaskContext): Iterator[U] =
    f(context, split.index, firstParent[T].iterator(split, context))
  override def clearDependencies() {
    super.clearDependencies()
    prev = null
  }
}

```

可以看出在MapPartitionsRDD里面都用到一个firstParent函数。仔细看一下，可以发现，在MapPartitionsRDD其实没有重写partition和compute逻辑，只是从firstParent中取了出来。

那么firstParent是干什么的呢？其实是取到父依赖。

```
  /** Returns the first parent RDD */
  protected[spark] def firstParent[U: ClassTag]: RDD[U] = {
    dependencies.head.rdd.asInstanceOf[RDD[U]]
  }
```

**注意：** 不要忽略细节。不然不太容易理解。

现在再看一下MapPartitionsRDD继承的RDD，它继承的是`RDD[U](prev)`。 这里的prev其实指的就是我们的HadoopRDD，也也就是说HadoopRDD变成了这个MapPartitionsRDD的OneToOneDependency依赖。OneToOneDependency是窄依赖。

```
def this(@transient oneParent: RDD[_]) =
    this(oneParent.context , List(new OneToOneDependency(oneParent)))
```

**总结：** 至此，我们阅读了第一行代码背后涉及的源码。`val textFile = sc.textFile("hdfs://...")
`，我们继续进行。不要急，后面会快很多。



### 4. 源码分析：`flatMap`方法、`filter`方法

接下面看一下flatMap、filter和map操作，观察一下下面的代码，其实他们都是返回了MapPartitionsRDD对象，不同的仅仅是传入的function不同而已。


经过前面的分析我们也可以知道，这些都是窄依赖。

```
  /**
   *  Return a new RDD by first applying a function to all elements of this
   *  RDD, and then flattening the results.
   */
  def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.flatMap(cleanF))
  }
```

```
 /**
   * Return a new RDD containing only the elements that satisfy a predicate.
   */
  def filter(f: T => Boolean): RDD[T] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[T, T](
      this,
      (context, pid, iter) => iter.filter(cleanF),
      preservesPartitioning = true)
  }
```


**注意：** 这里，我们可以明白了MapPartitionsRDD的compute方法的作用了：

1. 在没有依赖的条件下，根据分片的信息生成遍历数据的Iterable接口
2. 在有前置依赖的条件下，在父RDD的Iterable接口上给遍历每个元素的时候再套上一个方法

### 5. 源码分析：`PairRDDFunctions` 类

接下来，该reduceByKey操作了。它在PairRDDFunctions里面。

reduceByKey稍微复杂一点，因为这里有一个同相同key的内容聚合的一个过程，它调用的是combineByKey方法。

```
/**
   * Merge the values for each key using an associative reduce function. This will also perform
   * the merging locally on each mapper before sending results to a reducer, similarly to a
   * "combiner" in MapReduce.
   */
  def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)] = self.withScope {
    combineByKeyWithClassTag[V]((v: V) => v, func, func, partitioner)
  }

```

下面详细看一下combineByKeyWithClassTag。英文的注释写的挺清晰的，不再做讲解了。只提一点，这个操作，最后会new一个ShuffledRDD，然后调用它的一些方法，下面专门分析一下这个步骤。


>Generic function to combine the elements for each key using a custom set of aggregation functions. Turns an RDD[(K, V)] into a result of type RDD[(K, C)], for a "combined type" C (Int, Int) into an RDD of type (Int, Seq[Int]). Users provide three functions:

> - `createCombiner`, which turns a V into a C (e.g., creates a one-element list)
> - `mergeValue`, to merge a V into a C (e.g., adds it to the end of a list)
> - `mergeCombiners`, to combine two C's into a single one.

> In addition, users can control the partitioning of the output RDD, and whether to perform map-side aggregation (if a mapper can produce multiple items with the same key).

```
def combineByKeyWithClassTag[C]( createCombiner: V => C,mergeValue: (C, V) => C,mergeCombiners: (C, C) => C,partitioner: Partitioner,mapSideCombine: Boolean = true,serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)] = self.withScope {
    require(mergeCombiners != null, "mergeCombiners must be defined") // required as of Spark 0.9.0
    // 判断keyclass是不是array类型，如果是array并且在两种情况下throw exception。
    if (keyClass.isArray) {
      if (mapSideCombine) {
        throw new SparkException("Cannot use map-side combining with array keys.")
      }
      if (partitioner.isInstanceOf[HashPartitioner]) {
        throw new SparkException("Default partitioner cannot partition array keys.")
      }
    }
    val aggregator = new Aggregator[K, V, C](
      self.context.clean(createCombiner),
      self.context.clean(mergeValue),
      self.context.clean(mergeCombiners))
    //虽然不太明白，但是此处基本上一直是false，感兴趣的看后面的参考文章
    if (self.partitioner == Some(partitioner)) {
      self.mapPartitions(iter => {
        val context = TaskContext.get()
        new InterruptibleIterator(context, aggregator.combineValuesByKey(iter, context))
      }, preservesPartitioning = true)
    } else {
      // 默认是走的这个方法
      new ShuffledRDD[K, V, C](self, partitioner)
        .setSerializer(serializer)
        .setAggregator(aggregator)
        .setMapSideCombine(mapSideCombine)
    }
  }
```

### 6. 源码分析：`ShuffledRDD` 类

看一下上一段代码最后做了什么？这里传入了partitioner，并分别set了三个值。

```
new ShuffledRDD[K, V, C](self, partitioner)
        .setSerializer(serializer)
        .setAggregator(aggregator)
        .setMapSideCombine(mapSideCombine)
```

shuffle的过程有点复杂，先不深入讲解，后面专门来分析。这里先看一下依赖的关系ShuffleDependency，它是一个宽依赖。

```
  override def getDependencies: Seq[Dependency[_]] = {
    List(new ShuffleDependency(prev, part, serializer, keyOrdering, aggregator, mapSideCombine))
  }
```

**总结：** 至此，我们大致理了一遍RDD的转换过程，其实到现在为止还都是RDD的变换，还没有真正的执行，真正的执行会在最后一句的地方出发。

## 0x03 总结

本来在这篇博客中是想把RDD的shuffle的原理也写清楚，但是错估了一些工作量，前面的东西从学习整理到写出来就画了5个多小时。有点累了，加上已经10点半了，准备休息。 下次会专门讲清楚。

***

2017-05-21 22:34:00 wxxy

## 参考

- Matei Zaharia's paper
- http://spark.apache.org/
- http://www.cnblogs.com/cenyuhai/p/3779125.html
- http://www.infoq.com/cn/articles/spark-core-rdd/
- http://bit1129.iteye.com/blog/2178322
