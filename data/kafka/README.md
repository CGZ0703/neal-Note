# Kafka

![](../../.gitbook/assets/data/kafka/kafka-apis.png)

> Kafka® is used for building real-time data pipelines and streaming apps. It is horizontally scalable, fault-tolerant, wicked fast.

## Kafka是一个分布式的数据流平台

#### 流平台具有三个关键功能：

1. 发布和订阅 消息的流，类似于消息队列或企业消息传递系统
2. 以容错的持久化的方式存储消息
3. 当产生消息时进行处理

#### Kafka经常被用作两大类应用：

1. 在系统和应用程序之间构建可靠的实时流式数据通道
2. 构建可以转换或相应数据流的实时流式计算应用

### 基本概念

1. Kafka可以运行在几个跨多数据中心服务器的集群上
2. Kafka集群把消息(record)存储在被叫做主题(topic)的分类中
3. 每条消息都由一个key，一个value，一个timestamp组成

## 组件介绍

Broker：缓存代理，Kafka集群中的一台或多台服务器统称为broker。一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic

Producers：消息和数据生产者，向Kafka的一个topic发送消息的过程叫做producers（producer可以选择向topic哪一个partition发送数据）

Consumers：消息和数据消费者，接收topics并处理其发布的消息的过程叫做consumer，同一个topic的数据可以被多个consumer接收

Consumer Group （CG）：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partion只会把消息发给该CG中的一个consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。

Topic：特指Kafka处理的消息源的不同分类，其实也可以理解为对不同消息源的区分的一个标识

Partition：Topic物理上的分组，一个topic可以设置为多个partition，每个partition都是一个有序的队列，partition中的每条消息都会被分配一个有序的id（offset）

Message：消息，是通信的基本单位，每个producer可以向一个topic（主题）发送一些消息


### Topic

topic是kafka发送消息的一个标识，一般以目录的形式存在，对于一个有三个partition的topic而言，它日志信息结构大概如下图所示：

![](../../.gitbook/assets/data/kafka/log_anatomy.png)

每一个partition实际上都是一个有序的，不可变的消息序列，producer发送到broker的消息会写入到相应的partition目录下，每个partition都会有一个有序的id（offset），这个offset确定这个消息在partition中的具体位置。

![](../../.gitbook/assets/data/kafka/log_consumer.png)

当启动producer程序时，就会向kafka集群发送信息，而kafka就会把中间信息存储在这三个目录下。


### Producer

producer就是把消息发送给它所选择的topic，也可以具体指定发给这个topic的哪个一个partition，否则producer就会使用hashing-based partitioner来决定发送到哪个partition，这个问题还是需要多说一些，之前我在测试kafka速度的时候就遇到了这个问题，当我们增加broker的数量时，kafka的发送速度并没有线性增加，最后发现就是因为这个原因，没有指明发送数据到哪个partition。


### Consumer & Consumer Group

consumer是一个抽象的概念，调用Consumer API的程序都可以称作为一个consumer，它从broker端订阅某个topic的消息。如果只有一个consumer的话，该topic（可能含有多个partition）下所有消息都会被这个consumer接收。但是在分布式的环境中，我们可能会遇到这样一种情景，对于一个有多个partition的topic，我们希望启动多个consumer去消费这些partition（如果发送速度较快，一个consumer是无法消费完的），并且要求topic的一条消息只能发给其中一个consumer，不希望这些conusmer出现重复接收一条消息的情况。对于这种情况，我们应该怎么办呢？kafka给我们提供了一种机制，可以很好来适应这种情况，那就是consumer group（当然也可以应用在第一种情况，实际上，如果只有一个consumer时，是不需要指定consumer group，这时kafka会自动给这个consumer生成一个group名）。

在调用conusmer API时，一般都会指定一个consumer group，该group订阅的topic的每一条消息都发送到这个group的某一台机器上。假如kafka集群有两台broker，集群上有一个topic，它有4个partition，partition 0和1在broker1上，partition 2和3在broker2上，这时有两个consumer group同时订阅这个topic，其中一个group有2个consumer，另一个consumer有4个consumer，则它们的订阅消息情况如下图所示：

![](../../.gitbook/assets/data/kafka/consumer-groups.png)

因为group A只有两个consumer，所以一个consumer会消费两个partition；而group B有4个consumer，一个consumer会去消费一个partition。这里要注意的是，kafka可以保证一个partition内的数据是有序的，所以group B中的consumer收到的数据是可以保证有序的，但是Group A中的consumer就无法保证了。

group读取topic，partition分配机制是：

如果group中的consumer数小于topic中的partition数，那么group中的consumer就会消费多个partition；
如果group中的consumer数等于topic中的partition数，那么group中的一个consumer就会消费topic中的一个partition；
如果group中的consumer数大于topic中的partition数，那么group中就会有一部分的consumer处于空闲状态。
