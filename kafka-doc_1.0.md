kafka doc
#1. GETTING STARTED
##    1. Introduction
### kafka介绍

kafka是一个分布式的数据流平台。
数据流平台有以下三个方面的能力：
1. 可以发布订阅流式记录。从某些方面看，可以认为是消息队列或者是企业消息系统
2. 以一种容错的方式存储流式记录
3. 生成数据后，实时处理流式数据

kafka 适用于两种类型的应用：
1. 建立一个在系统或应用之间实时获取数据的数据流管道；
2. 建立一个实时处理流式数据的流式应用

为了了解kafka是如何实现的，让我们深入了解kafka的能力。

首先有一些概念：
1. kafka在一个或者多个server的集群上运行；
2. kafka集群存储流式数据以topic分类存储
3. 每条记录都由key,value和timestamp;
4. Api:
- Producer API：发布流式记录 
- Consumer API：订阅流式记录，并处理 
- Streams API： 让应用成为stream的一个处理器，从一个或者多个topic中消费输入流，并生成输出流到一个或者多个topic中，高效的将输入流转换到输出流；
- Connector API：允许建立运行一个可以重用的生产者和消费者，连接topic到数据系统。例如：可以连接到关系数据库，检测每个表的变化。

client和server的交互使用一个简单，高性能的Tcp协议。这个协议和老版本兼容。

###Topics and Logs

每个topic,kafka都管理一个分区的log.每个分区都是有区无法修改的记录序列，并且持续在追加。记录在分区中的位置叫offset.
kafka集群会保存记录一段时间，无论是否被消费。
offset可以实现消费者灵活消费数据，而不影响其他的consumer。
分区有两个目的：
- 可以实现日志的水平扩展，从而使topic可存储的数据量超过单台机器的数据量；
- 作为并发的单元，
##分布式
分区数据部署在集群中的机器中，每台机器都可以处理分区的请求以及数据。每个分区都会复制一定数量到多台机器中，实现容错。

每个分区都有一个leader和0到多个follower。leader处理所有的分区读写请求，follower被动同步leader。如果leader发生故障，一个follower会自动变成新的leader。一个server会为它的分区作为一个leader，其他的为follower，因此集群很好进行了负载。

##producer

Producer发布数据到topic，producer可以自定义选择将数据发布到指定topic的指定分区。可以通过round-robin或者其他算法来实现均衡。

##Consumers

Consumers会给他们命名一个组名，并且每条记录都会发布到定阅消息的group的一个consumer实例中。

如果所有的Consumer 实例有同一个group，那么所有的消息都会负载均衡。

如果所有的Consumer实例有不同的group，那么每条消息都会广播到所有的Consumer 进程。

一般情况下，我们发现topic有一些consumer的group，每个group都是逻辑的消息订阅者。每个group都由很多的Consumer 实例组成。这不是普通的发布订阅模型算法，订阅者由许多的consumer组成，而不是一个单独的进程。
通过分区来实现每个Consumer 实例在同一个时间点只会唯一消费一个分区数据。kafka会维护Consumer group中的成员，分配分区。如果有新的Consumers实例加入或者退出，会将分区进行重新分配。

kafka只有提供分区有序，而无法保证一个topic内的不同分区有序。每个分区的有序性通过key来实现，对于大多数应用是足够的。然后，如果需要全部数据有序，只能是一个topic一个分区来实现，同时只能是一个group一个Consumer实例。

###Guarantees

high-level kafka有几下的保证：
- 同一个producer发送到指定topic分区的消息，会按发送的顺序保存到kafka中。
- 消息者收到消息的顺序与保存的顺序一致
- 一个topic有N个副本，那么可以容忍N-1个server失效，而不会丢失数据。
- 

##kafka作为消息系统

传统的消息系统有两种模型：队列和发布订阅。队列是很多consumer都可以从队列中读取消息，并且其中消息会被其中一个consumer消费。发布订阅模型，消息会被广播给所有的消费者。两种都有优缺点：队列的优点：消息处理可以水平扩展到多个consumer实例中；然后无法实现多个订阅者，一条消息处理完以后，其他consumer无法读取。发布订阅可以发布数据到每个consumer中，但是无法实现水平扩展，每个消息只能对应一个消费者。

kafka中的consumer group解决了以上问题。
kafka比传统的消息系统，有很强的顺序保证。
topic中的分区，kafka可以提供有序性以及consumer group中的负载均衡。每个分区分配一个consumer，可以保证消费数据的有序性，很多分区可能实现很多消费者的负载均衡。

## Kafka as 存储系统

可以把kafka当做一种特殊的分布式文件系统，具有高性能，低延迟，多副本，传播。


##Kafka 流式处理

仅仅读写，存储流式数据是不够的，目标是通过实时处理数据流。
kafka中的流式处理是不断的从kafka topic中获取数据，处理后，产生不断的数据流。

简单处理可以直接使用producer或者conusmerAPI。然后，一些复杂的处理，kafka提供了stream api。允许基于流来构建应用。
stream api是基于原始的kafka概念来实现的： producer and consumer APIs来输入，kafka来存储，group来实现容错。

###各个功能的结合


##1.2 Use Cases

###消息队列

kafka是一个很好的传统的消息队列的替代品。
可以替代： ActiveMQ or RabbitMQ.

###网站活动追踪

kafka最初使用的案例就是建立一个用户行为追踪的的数据流，每个页面都会生成大量的活动数据。

###metrics

收集分布式系统的数据，汇集成一个中心的运营数据。

### 日志汇集

### 流式处理

从0.10发布了流式处理库 Kafka Streams 。
Apart from Kafka Streams, alternative open source stream processing tools include Apache Storm and Apache Samza.

### 事件溯源

存储一系列基于时间的状态变化记录。

###commit log

kafka可以做为分布式外部commit log。commit log可以帮助分布式的结点进行数据同步复制，以及故障恢复。这种使用情况下，kafka类似Apache BookKeeper project.

## 1.3 Quick Start

- Step 7: Use Kafka Connect to import/export data

可以使用kafka Connect来实现导入导出数据，而无须编写定制化的集成代码；

- Step 8: Use Kafka Streams to process data
kafka streams 是一个客户端库用来实现实时性要求非常高的应用或者微服务，数据的输入和输出都存储在kafka 集群中。



















