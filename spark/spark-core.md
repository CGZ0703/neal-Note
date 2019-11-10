# Spark 基础

## RDD

### 定义

- 数据集：存储的数据的计算逻辑
- 分布式：数据的来源，计算，存储
- 弹  性：
  1. 血缘：依赖关系，Spark可以通过特殊的处理方案简化的依赖关系
  2. 计算：Spark的计算是基于内存的，所以性能特别高，可以和磁盘灵活切换
  3. 分区：Spark在创建默认分区后，可以通过指定的算子来改变分区数量
  4. 容错：Spark在执行计算时，如果发生了错误，需要进行容错重试处理
- 数  量：
  1. Executor：可以通过提交应用的参数进行设定
  2. Partition：默认情况下，读取文件采用的是Hadoop的切片规则，如果读取内存中的数据，可以根据特定的算法进行设定，可以通过其他算子进行改变。多个阶段的场合，下一个阶段的分区数量取决于上一个阶段最后RDD的分区数，但是可以在相应的算子中可以修改
  3. Stage：ResultStage(1)+ShuffleMapStage(Shuffle依赖的数量)。划分阶段的目的就是为了任务执行的等待，因为Shuffle的过程需要落盘
  4. Task：原则上一个分区就是一个任务，但是实际应用中，可以动态调整，一般为CPU内核的3-5倍。


### 创建

- 从内存中创建
- 从存储中创建
- 从其他RDD创建


### 属性

- 分区
- 依赖关系
- 分区器
- 优先位置
- 计算函数


### 使用

- 转换
  - 单 Value 类型
    1. map(func)
    2. mapPartitions(func)
    3. flatMap(func)
    4. glom()
    5. groupBy(func)
    6. filter(func)
    7. sample(withReplacement, fraction, seed)
    8. distinct(\[numPartitions\])
    9. coalesce(numPartitions)
    10. repartition(numPartitions)
    11. sortBy(func, \[ascending\])
    12. pipe(command, \[envMap\])
  - 双 Value 类型
    1. union(otherDataset)
    2. subtract(otherDataset)
    3. intersection(otherDataset)
    4. cartesian(otherDataset)
    5. zip(otherDataset)
  - K-V 类型
    1. partitionBy
    2. groupByKey
    3. reduceByKey
    4. aggregateByKey
    5. foldByKey
    6. combineByKey
    7. mapValues
    8. join(otherDataset, \[numPartitions\])
    9. cogroup(otherDataset, \[numPartitions\])
- 行动：runJob
  1. reduce(func)
  2. collect()
  3. count()
  4. first()
  5. take(num)
  6. takeOrdered(num)
  7. aggregate
  8. fold(num)(func)
  9. saveAsTextFile(path)
  10. saveAsSequenceFile(path)
  11. saveAsObjectFile(path)
  12. countByKey()
  13. foreach(func)


## 广播变量

分布式共享只读数据，对于一份多次使用的变量来说广播给每个Executor使用。例如字典对应表的数据


## 累加器

```scala
/**
 * Returns if this accumulator is zero value or not.
 * e.g. for a counter accumulator, 0 is zero value;
 *      for a list accumulator, Nil is zero value.
 */
def isZero: Boolean  
/**
 * Creates a new copy of this accumulator.
 */
def copy(): AccumulatorV2[IN, OUT]  
/**
 * Resets this accumulator, which is zero value.
 * i.e. call `isZero` must return true.
 */
def reset(): Unit  
/**
 * Takes the inputs and accumulates.
 */
def add(v: IN): Unit  
/**
 * Merges another same-type accumulator into this one and update its state,
 * i.e. this should be merge-in-place.
 */
def merge(other: AccumulatorV2[IN, OUT]): Unit  
/**
 * Defines the current value of this accumulator
 */
def value: OUT  
```

```scala
val myAccumulator = new MyAccumulator
sc.register(myAnnumulator)
