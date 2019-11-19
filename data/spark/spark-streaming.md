# Spark Streaming

## 实时流数据

## DStream - Discretized Stream

DStream 和 RDD 之间的关系。

DStream代表了一系列连续的RDD，DStream中每个RDD包含特定时间间隔的数据，每隔一段固定时间（可设置）生成一个批次的RDD，于DStream原语定义的操作所进行处理。

一个DStream对应了时间维度上的多个RDD。在一个固定的维度上，DStream和RDD是一一对应关系，可以将DStream看成是RDD在时间维度上封装。

DStream的操作和RDD的操作惊人的相似，通过对DStream的不断转换，形成依赖关系。所有的DStream操作最终会转换成底层的RDD操作。最后spark程序由于执行了foreachRDD算子中的RDD操作被触发。

- DStream 中有个属性generatedRDDs，其中，Key为作业开始时间，Value为该时刻对应产生的RDD：
```scala
DStream.scala

  // RDDs generated, marked as private[streaming] so that testsuites can access it
  @transient
  private[streaming] var generatedRDDs = new HashMap[Time, RDD[T]]()
```


### Input DStreams and Receivers

`Input DStream`是接收数据源生成的DStream，每个`Input DStream`都与一个`Receiver`对象关联，以从数据源接收数据并存储到Spark的内存中等待处理。

Spark Streaming 提供了两类内置的流媒体源：
- Basic sources：直接在StreamingContext API中可用的来源。示例： file systems和 socket connections。
- Advanced sources：可以通过其他实用程序类获得。如：Kafka，Flume，Kinesis等资源。需要针对额外的依赖项进行连接。

**注意**

本地执行Spark Streaming程序时，master URL不要设置为`local`或`local[1]`，这两种方式意味着仅一个线程将用于本地运行任务，如果使用基于receiver的input DStream(例如: sockets, Kafka, Flume, etc.)，则将会使用唯一的线程来运行receiver，而不会留下任何线程来处理接收到的数据。因此，请使用`local[n]`作为master URL，其中 `n > 运行的receiver`。

同样，在集群上运行时，分配给Spark Streaming应用程序的核心数必须大于receiver的数量，否则，系统将只接收数据，但无法处理它。

#### socket Input DStream
ssc.socketTextStream方法生成了`SocketInputDStream => ReceiverInputDStream => InputDStream`

- DStream 定义了compute，调用了 ReceiverInputDStream中实现的 compute
```scala
   // Generates RDDs with blocks received by the receiver of this stream.
=> compute(validTime: Time)
   // Create the BlockRDD
=> val blockInfos = receiverTracker.getBlocksOfBatch(validTime).getOrElse(id, Seq.empty)
=> createBlockRDD(validTime, blockInfos)
   // Else, create a BlockRDD. However, if there are some blocks with WAL info but not others then that is unexpected and log a warning accordingly.
   // If no block is ready now, creating WriteAheadLogBackedBlockRDD or BlockRDD according to the configuration
=> val validBlockIds = blockIds.filter { id => ssc.sparkContext.env.blockManager.master.contains(id) }
=> new BlockRDD[T](ssc.sc, validBlockIds)
 or
=> new BlockRDD[T](ssc.sc, Array.empty)
```

#### Kafka Input DStream

|  | spark-streaming-kafka-0-8 | spark-streaming-kafka-0-10 |
| --- | --- | --- |
| Broker Version | 0.8.2.1 or higher | 0.10.0 or higher |
| Api Stability | Stable | Experimental |
| Language Support | Scala, Java, Python | Scala, Java |
| Receiver DStream | Yes | No |
| Direct DStream | Yes | Yes |
| SSL / TLS Support | No | Yes |
| Offset Commit Api | No | Yes |
| Dynamic Topic Subscription | No | Yes |


### Transformations on DStreams

类似 RDD 中的 转换算子

```scala
new TransformedDStream(parent: DStream,transformedFunc)
   // DStream.getOrComput()
=> parent.getOrCompute(validTime) ...
```

实际上是通过compute方法调用了parent的getOrComput方法获取了父DStream的RDD

通过对父DStream的RDD的操作生成新的RDD，转换的业务逻辑通过transformedFunc参数获得。

这样对DStream的操作都转换成了对RDD的操作，同时DStream的依赖关系也与RDD之间的依赖关系同时建立了起来

| Transformation | Meaning |
| --- | --- |
| map(func) | Return a new DStream by passing each element of the source DStream through a function func. |
| flatMap(func) | Similar to map, but each input item can be mapped to 0 or more output items. |
| filter(func) | Return a new DStream by selecting only the records of the source DStream on which func returns true. |
| repartition(numPartitions) | Changes the level of parallelism in this DStream by creating more or fewer partitions. |
| union(otherStream) | Return a new DStream that contains the union of the elements in the source DStream and otherDStream. |
| count() | Return a new DStream of single-element RDDs by counting the number of elements in each RDD of the source DStream. |
| reduce(func) | Return a new DStream of single-element RDDs by aggregating the elements in each RDD of the source DStream using a function func (which takes two arguments and returns one). The function should be associative and commutative so that it can be computed in parallel. |
| countByValue() | When called on a DStream of elements of type K, return a new DStream of (K, Long) pairs where the value of each key is its frequency in each RDD of the source DStream. |
| reduceByKey(func, [numTasks]) | When called on a DStream of (K, V) pairs, return a new DStream of (K, V) pairs where the values for each key are aggregated using the given reduce function. Note: By default, this uses Spark's default number of parallel tasks (2 for local mode, and in cluster mode the number is determined by the config property spark.default.parallelism) to do the grouping. You can pass an optional numTasks argument to set a different number of tasks. |
| join(otherStream, [numTasks]) | When called on two DStreams of (K, V) and (K, W) pairs, return a new DStream of (K, (V, W)) pairs with all pairs of elements for each key. |
| cogroup(otherStream, [numTasks]) | When called on a DStream of (K, V) and (K, W) pairs, return a new DStream of (K, Seq[V], Seq[W]) tuples. |
| transform(func) | Return a new DStream by applying a RDD-to-RDD function to every RDD of the source DStream. This can be used to do arbitrary RDD operations on the DStream. |
| updateStateByKey(func) | Return a new "state" DStream where the state for each key is updated by applying the given function on the previous state of the key and the new values for the key. This can be used to maintain arbitrary state data for each key. |


### Output Operations on DStreams

Output Operations允许将DStream的数据推出到外部系统，例如database或file system。由于Output Operations实际上使用转换后的数据导出到外部系统中，因此它们会触发所有DStream转换的实际执行（类似于RDD的actions）。

| Output Operation | Meaning |
| --- | --- |
| print() | Prints the first ten elements of every batch of data in a DStream on the driver node running the streaming application. This is useful for development and debugging. |
| saveAsTextFiles(prefix, [suffix]) | Save this DStream's contents as text files. The file name at each batch interval is generated based on prefix and suffix: "prefix-TIME_IN_MS[.suffix]". |
| saveAsObjectFiles(prefix, [suffix]) | Save this DStream's contents as SequenceFiles of serialized Java objects. The file name at each batch interval is generated based on prefix and suffix: "prefix-TIME_IN_MS[.suffix]". |
| saveAsHadoopFiles(prefix, [suffix]) | Save this DStream's contents as Hadoop files. The file name at each batch interval is generated based on prefix and suffix: "prefix-TIME_IN_MS[.suffix]". |
| foreachRDD(func) | The most generic output operator that applies a function, func, to each RDD generated from the stream. This function should push the data in each RDD to an external system, such as saving the RDD to files, or writing it over the network to a database. Note that the function func is executed in the driver process running the streaming application, and will usually have RDD actions in it that will force the computation of the streaming RDDs. |

#### foreachRDD

foreachRDD中创建了一个ForeachDStream对象，就是Output DStream，调用了该对象的register方法，就爱那个当前对象注册给DStreamGraph，加入graph的输出流outputStream中。

其中，foreachRDD操作针对每个RDD进行操作，

当需要创建外部系统的连接并导出数据时，推荐使用以下方式

```scala
dstream.foreachRDD { rdd =>
  rdd.foreachPartition { partitionOfRecords =>
    // ConnectionPool is a static, lazily initialized pool of connections
    val connection = ConnectionPool.getConnection()
    partitionOfRecords.foreach(record => connection.send(record))
    ConnectionPool.returnConnection(connection)  // return to the pool for future reuse
  }
}
```

另： 中间还可加如检测，rdd或partitionOfRecords的数据存在性，以减少不必要的连接创建与释放。

## 触发RDD的执行

在 Spark Stream的执行过程中，有以下几个核心对象：
1. JobScheduler
2. JobGenerator
3. DStreamGraph
4. DStream















