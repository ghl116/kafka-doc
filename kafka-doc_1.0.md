kafka doc
# 1. GETTING STARTED
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

### Topics and Logs

每个topic,kafka都管理一个分区的log.每个分区都是有区无法修改的记录序列，并且持续在追加。记录在分区中的位置叫offset.
kafka集群会保存记录一段时间，无论是否被消费。
offset可以实现消费者灵活消费数据，而不影响其他的consumer。
分区有两个目的：
- 可以实现日志的水平扩展，从而使topic可存储的数据量超过单台机器的数据量；
- 作为并发的单元，
##分布式
分区数据部署在集群中的机器中，每台机器都可以处理分区的请求以及数据。每个分区都会复制一定数量到多台机器中，实现容错。

每个分区都有一个leader和0到多个follower。leader处理所有的分区读写请求，follower被动同步leader。如果leader发生故障，一个follower会自动变成新的leader。一个server会为它的分区作为一个leader，其他的为follower，因此集群很好进行了负载。

## producer

Producer发布数据到topic，producer可以自定义选择将数据发布到指定topic的指定分区。可以通过round-robin或者其他算法来实现均衡。

## Consumers

Consumers会给他们命名一个组名，并且每条记录都会发布到定阅消息的group的一个consumer实例中。

如果所有的Consumer 实例有同一个group，那么所有的消息都会负载均衡。

如果所有的Consumer实例有不同的group，那么每条消息都会广播到所有的Consumer 进程。

一般情况下，我们发现topic有一些consumer的group，每个group都是逻辑的消息订阅者。每个group都由很多的Consumer 实例组成。这不是普通的发布订阅模型算法，订阅者由许多的consumer组成，而不是一个单独的进程。
通过分区来实现每个Consumer 实例在同一个时间点只会唯一消费一个分区数据。kafka会维护Consumer group中的成员，分配分区。如果有新的Consumers实例加入或者退出，会将分区进行重新分配。

kafka只有提供分区有序，而无法保证一个topic内的不同分区有序。每个分区的有序性通过key来实现，对于大多数应用是足够的。然后，如果需要全部数据有序，只能是一个topic一个分区来实现，同时只能是一个group一个Consumer实例。

### Guarantees

high-level kafka有几下的保证：
- 同一个producer发送到指定topic分区的消息，会按发送的顺序保存到kafka中。
- 消息者收到消息的顺序与保存的顺序一致
- 一个topic有N个副本，那么可以容忍N-1个server失效，而不会丢失数据。
- 

## kafka作为消息系统

传统的消息系统有两种模型：队列和发布订阅。队列是很多consumer都可以从队列中读取消息，并且其中消息会被其中一个consumer消费。发布订阅模型，消息会被广播给所有的消费者。两种都有优缺点：队列的优点：消息处理可以水平扩展到多个consumer实例中；然后无法实现多个订阅者，一条消息处理完以后，其他consumer无法读取。发布订阅可以发布数据到每个consumer中，但是无法实现水平扩展，每个消息只能对应一个消费者。

kafka中的consumer group解决了以上问题。
kafka比传统的消息系统，有很强的顺序保证。
topic中的分区，kafka可以提供有序性以及consumer group中的负载均衡。每个分区分配一个consumer，可以保证消费数据的有序性，很多分区可能实现很多消费者的负载均衡。

## Kafka as 存储系统

可以把kafka当做一种特殊的分布式文件系统，具有高性能，低延迟，多副本，传播。


## Kafka 流式处理

仅仅读写，存储流式数据是不够的，目标是通过实时处理数据流。
kafka中的流式处理是不断的从kafka topic中获取数据，处理后，产生不断的数据流。

简单处理可以直接使用producer或者conusmerAPI。然后，一些复杂的处理，kafka提供了stream api。允许基于流来构建应用。
stream api是基于原始的kafka概念来实现的： producer and consumer APIs来输入，kafka来存储，group来实现容错。

### 各个功能的结合


## 1.2 Use Cases

### 消息队列

kafka是一个很好的传统的消息队列的替代品。
可以替代： ActiveMQ or RabbitMQ.

### 网站用户行为数据追踪

kafka最初使用的案例就是建立一个用户行为追踪的的数据流，每个页面都会生成大量的活动数据。

### metrics

收集分布式系统的数据，汇集成一个中心的运营数据。

### 日志汇集

### 流式处理

从0.10发布了流式处理库 Kafka Streams 。
Apart from Kafka Streams, alternative open source stream processing tools include Apache Storm and Apache Samza.

### 事件溯源

存储一系列基于时间的状态变化记录。

### commit log

kafka可以做为分布式外部commit log。commit log可以帮助分布式的结点进行数据同步复制，以及故障恢复。这种使用情况下，kafka类似Apache BookKeeper project.

## 1.3 Quick Start

- Step 7: Use Kafka Connect to import/export data

可以使用kafka Connect来实现导入导出数据，而无须编写定制化的集成代码；

- Step 8: Use Kafka Streams to process data

kafka streams 是一个客户端库用来实现实时性要求非常高的应用或者微服务，数据的输入和输出都存储在kafka 集群中。kafka stream具有开发部署简单优点，可以使用java和scala开发，具有高可扩展，易用，容错，分布式等特性。
- demmo : http://kafka.apache.org/10/documentation/streams/quickstart


## 1.4 kafka 生态

有很多和kafka集成的工具。
https://cwiki.apache.org/confluence/display/KAFKA/Ecosystem

## 1.5 升级

`（可以看看cdh的升级方案）`
### Upgrading from 0.8.x, 0.9.x, 0.10.0.x, 0.10.1.x, 0.10.2.x or 0.11.0.x to 1.0.0

推荐使用滚动升级的方式，实现服务不间断。

滚动升级：
1. 修改所有broker的配置文件server.properties，增加以下内容
> inter.broker.protocol.version=CURRENT_KAFKA_VERSION (e.g. 0.8.2, 0.9.0, 0.10.0, 0.10.1, 0.10.2, 0.11.0).
> log.message.format.version=CURRENT_MESSAGE_FORMAT_VERSION 

2. 一次升级一台broker：停止服务，升级程序，启动；
3. 整个集群升级完成后，修改协议版本为：inter.broker.protocol.version = 1.0
4. 滚动重启broker，生效；
5. 如果已经覆盖了消息格式版本，需要再进行一次滚动重启。
    -  如果所有的consumer已经升级到0.11版本以上，将每个broker的的log.message.format.version 属性改成 1.0  ，然后滚动重启；
    -  老版本的scala consumer不支持 0.11引入的格式，新版本的java consumer一定要使用。

其他升级提示：

1. 也可以进行停机升级，重启后，默认使用的是新的协议；
2. broker升级后，协议版本和消息格式版本，并重启生效可以任何时间进行。

### 1.0.0 主要变更

- 删除topic功能目前是默认启用。
- topic支持timestame搜索，不包含offset的分区也会被搜索到；
- inter.broker.protocol.version 1.0以后的版本，当有log目录不可用时，broker状态仍然可用。用户需要监控metrics:offlineLogDirectoryCount,来判断是否有目录不可用；
- 增加了KafkaStorageException
- JVM默认配置使用了 -XX:+ExplicitGCInvokesConcurrent替换-XX:+DisableExplicitGC
- handleError的重写方法已经被移除；
- java 客户端和工具接受string类型的client-id
- kafka-consumer-offset-checker.sh已删除，使用kafka-consumer-groups.sh替换
- SimpleAclAuthorizer现在默认会对拒绝授权的日志记录信息；
- 认证失败会提示客户端错误信息为：AuthenticationException的子类；认证失败不会进行重试；
- 实现定制化的SaslServer在认证失败时，需要抛出SaslAuthenticationException。
- JMX中的app-info mbean被metrics取代了；
- Kafka metrics可以包含非数值参数；org.apache.kafka.common.Metric#metricValue()
- 每个kafka处理速率的metric都有对应的一个总量，比如：records-consumed-rate 和 records-consumed-total
- kafka_mx4jenable=true，启用Mx4j
- org.apache.kafka.common.security.auth 这个包里面的类变成了公共的，并增加到文档中；
- 无授权但topic存在时，会返回TOPIC_AUTHORIZATION_FAILED 有授权，但是topic不存在时，报错：UNKNOWN_TOPIC_OR_PARTITION
- config/consumer.properties文件需要更新

### 升级 1.0.0 Kafka Streams Application

###Upgrading from 0.8.x, 0.9.x, 0.10.0.x, 0.10.1.x or 0.10.2.x to 0.11.0.0

###  0.11.0.0变化

- Unclean leader election默认禁用
- Producer 的配置参数 block.on.buffer.full, metadata.fetch.timeout.ms and timeout.ms被删除了
- broker配置的offsets.topic.replication.factor，在创建topic时会自动启用。如果副本数不够，创建topic会报错；
- 使用snappy压缩数据时，producer 和 broker提高了压缩率；一个producer使用5000个分区时，会使用315MB JVM
- 提高了gzip的压缩率
- broker的max.message.bytes属性现在应用到批量消息大小。
- GC log现在默认是滚动日志；
- RecordMetadata, MetricName and Cluster 类的构造函数已经删除了
- 增加 user headers，提供读写访问；
- ProducerRecord 和 ConsumerRecord 提供new Headers API 
- ExtendedSerializer and ExtendedDeserializer 提供用来序列化header
- 新配置参数group.initial.rebalance.delay.ms，默认3秒
- 如果topic不存在，查询元数据，org.apache.kafka.common.Cluster#partitionsForTopic, partitionsForNode and availablePartitionsForTopic 这些方法会返回一个空的数组，而不是null;
- streams API default.timestamp.extractor, default.key.serde, and default.value.serde 替换timestamp.extractor, key.serde, and value.serde
- Java consumer's commitAsync 提供offset失败时，返回信息优化

### 精确一次算法

kafka0.11,producer中支持幂等和事务。幂等传输确保一个producer的消息只会精确传输一次到一个topic的分区。事务传递实现了producer发送数据到多个分区，要么全部成功，要么全部失败；两个功能合在一起就是kafka的 精确传输一次。
- 只有新的java producer and consumer支持精确一次算法
- 这些特性需要依赖0.11消息格式；老版本的消息格式下使用该特性会报错；
- 事务状态被存储在内部的topic中__transaction_state，这个topic和consumer offset topic类似。这个topic只有使用事务功能时才会被创建。
- 为了安全考虑，事务的API需要依赖新的ACL（bin/kafka-acls.sh）
- EoS引入了几个新的API
 
## Notes on the new message format in 0.11.0

 To get around these problems, you should ensure 1) that the producer's batch size is not set larger than max.message.bytes, and 2) that the consumer's fetch size is set at least as large as max.message.bytes.
 In order to avoid the cost of down-conversion, you should ensure that consumer applications are upgraded to the latest 0.11.0 client.

### Upgrading from 0.8.x, 0.9.x, 0.10.0.x or 0.10.1.x to 0.10.2.0

### Notable changes in 0.10.2.1

- 修改了两个kafka stream的配置参数的默认值，retries（0- 10），max.poll.interval.ms （300000 to Integer.MAX_VALUE）

### Notable changes in 0.10.2.0

- Java clients (producer and consumer) 可以读写老版本的broker.
- 线程中断后，Java consumer会抛异常InterruptException
- Java consumer可以关闭连接；一个新的close API已增加到kafkaconsumer来控制最大等待时间。
- new Java consumer实现MirrorMaker，可以接受多个表达式；
- 升级stream应用， 0.10.1 to 0.10.2 ，不需要升级broker
- Streams API中删除了zookeeper的依赖
- StreamsConfig 增加了参数："security.protocol", "connections.max.idle.ms", "retry.backoff.ms", "reconnect.backoff.ms" and "request.timeout.ms" 


### Upgrading from 0.8.x, 0.9.x or 0.10.0.X to 0.10.1.0

### 在0.10.1.0版本中的重大变化

- 日志保留时间不再基于log segments的上次修改时间；而是基于最大时间戳
- 日志滚动时间不再依赖log segment创建时间。现在基于消息中的时间戳。例如：第一个消息的时间戳是T，日志会滚动当有一个消息的时间戳大于 T +  log.roll.ms
-  0.10.0的open file 打开数会增加 33%，因为每个segment的额外时间index
-  time index和 offset index会共享index 大小的配置。 由于time index 是1.5倍的offset index的大小。所以需要增加log.index.size.max.bytes，来避免滚动日志问题；
-  由于增加了index文件，broker启动时，加载log的时间会久一些。根据测试，num.recovery.threads.per.data.dir 设置成1，可以减少加载时间。


### Notable changes in 0.10.1.0

- 新版java consumer不再是beta版本，我们推荐所有的开发环境。scala consumer任然支持，再是会在后续版本移除掉。
- 使用新consumer的MirrorMaker and the Console Consumer 不需要再进行切换。
- kafka集群可以通过cluster id 来唯一标识。当升级到0.10.1.0时，这个标识会自动生成。
- broker状态RunningAsController，已经被删除了。
- new Java Consumer支持在分区中搜索时间戳
- new Java Consumer可以独立线程进行心跳测试。增加了新配置max.poll.interval.ms参数控制consumer离开group的超时时间。 The value of the configuration request.timeout.ms must always be larger than max.poll.interval.ms because this is the maximum time that a JoinGroup request can block on the server while the consumer is rebalancing, so we have changed its default value to just above 5 minutes
- 认证时，如果用户没有describe topic的权限，不会返回TOPIC_AUTHORIZATION_FAILED信息，会返回UNKNOWN_TOPIC_OR_PARTITION。
- 请求有大小限制，默认 (50 MB for consumers and 10 MB for replication). 
- Consumers and replicas can make progress if a message larger than the response/partition size limit is found.
- kafka.api.FetchRequest and kafka.javaapi.FetchRequest的构造函数中增加了指定分区顺序的参数；

### Upgrading from 0.8.x or 0.9.x to 0.10.0.0

### Potential breaking changes in 0.10.0.0

- 从0.10开始，消息格式代表了kafka版本；
- Message 格式 0.10被引入默认使用；
- 针对0.10，引入ProduceRequest/Response v2
- 引入FetchRequest/Response v2
- MessageFormatter接口改了
- MessageReader接口改了
- MessageFormatter 包名改了
- MessageReader 包名改了
- MirrorMakerMessageHandler类不再可见
- 0.7 KafkaMigrationTool被移除
- new consumer 标准化API，来接受java.util.Collection
- LZ4-compressed压缩处理使用框架：LZ4f v1.5.1

### Notable changes in 0.10.0.0

- 引入kafka stream
- receive.buffer.bytes有了默认值64K
- 新参数exclude.internal.topics被引入
- 老版本的Scala producer被弃用
- 新版本consumer已稳定
 
### Upgrading from 0.8.0, 0.8.1.X or 0.8.2.X to 0.9.0.0

### Potential breaking changes in 0.9.0.0
- Java 1.6不支持
- Scala 2.9 不支持
- broker id超过1000会保留分配给broker id.
- 参数replica.lag.max.messages被删除。
- 参数 replica.lag.time.max.ms不仅仅上次请求经过的时间，也指副本上次同事数据的时间；副本现在一直在从leader中获取数据，但是和最新的消息相差replica.lag.time.max.ms，这种情况被认为不同步；
- Compacted topics 不再接受没有key的消息
- MirrorMaker不支持多个目标集群


# 2. APIS

- The Producer API 
- The Consumer API 
- The Streams API 
- The Connect API ：不断把源系统数据传到kafka中或者把kafka数据传输到目标系统
- The AdminClient API ：管理监控topic,broker以及其他对象；
 
### 2.6 Legacy APIs
历史遗留的API仍然包含在kafka中，为了兼容。

# 3. 配置

# 4. 设计
## 4.1 动机
设计kakfa能够满足处理所有大公司实时数据的统一平台，为了实现，我们考虑了很多的案例。
- 需要能够有大的吞吐，来支持大量事件流的：如实时日志汇聚
- 可以处理大量数据，来实现定期从离线系统中加载数据；
- 低延迟的数据传递，可以处理更多的传统消息队列处理的场景；
- 当机器宕机时，还要保证容错
支持这些要求，需要我们设计很多，更类似数据库log，而不仅仅像传统的消息队列。

## 4.2 持久化

### 不要害怕文件系统

kafka 重度依赖文件系统实现存储和缓存消息。通常认为磁盘很慢，让人们怀疑它无法做一个高效的持久化存储。事实上看大家怎么用，使用得当，磁盘可以像网络一样快。


线性读写是被预测，并且被操作系统重度优化。现代操作系统提供预读数据，批量写操作。

现代操作系统提供了先读后写的技术，先预读数据，后续批量写数据；现代操作系统重度使用内存和磁盘缓存，来抵消随机读写的开消；当现代操作系统当使用内存时，会把内存的数据放到磁盘缓存中；所有的磁盘读写都会过这个缓存；这个特性是不会被关闭的，除非直接使用I/O；即使一个进程内存缓存了数据，数据也会被缓存到OS中。导致存储两遍；

此外，我们是基于JVM来进行开发的，有使用过java开发的人都应该知道两件事：
- 对象占用的内存是非常高的，常常是数据存储的两倍
- java内存回收会变得很慢而且很烦锁，当堆内存增长得很快的时候；

因此使用文件系统和pagecache要优于自己维护一个内存中的缓存；我至少访问内存时，会占用两倍的可用内存；存储时又会占用两倍；这会导致一台32G内存的机器，缓存占用28-30G；服务重启会，缓存会进行重建（10G缓存需要10分钟），或者重新初使化占用缓存。因此这表明通过操作系统来保证文件系统和缓存的一致性是合理的，也更加高效，而不是通过自己来尝试实现。如果你需要线性读取，那么使用缓存预读数据是非常有效的；

这表明了一个简单的设计：我们不是把数据保存在内存中，没有空间的时才把数据持久化到文件系统中；相反，所有的数据都会立刻持久化到文件系统中的一个log，而不是并要的时候才刷新到磁盘；事实上，这意味着我们数据是被写入到了内核的pagecache中。

``This style of pagecache-centric design is described in an article on the design of Varnish here (along with a healthy dose of arrogance).``

### 固定时间开消
消息队列中常见的持久化数据结构是每个用户一个BTtree的队列，以及其他随机访问的消息元数据的队列。BTree是非常强大的数据结构，并可以支持许多事务和非事务的消息队列，但是也有很高的开消，btree操作是O(logN).通常认为O(logN)是固定的处理时间，但是对于磁盘操作不是这样的，磁盘操作10ms一个pop，每个操作只能seek一次，无法并行 ；大量的磁盘查找会导致很高的开消。


一个关于简单读和追加到文件的持久消息队列是一个常见的解决方案。这种方案的优势是所有的操作是O（1），读操作不会阻塞写操作；另外一个明显的性能优势是：由于性能被数据量所减弱，server可以充分利用sata盘。虽然sata盘随机读写很慢，但是具有大量读写的可接受性能，而且只有1/3的价格，3倍的容量；

在无性能降低的情况下，访问无限容量的磁盘意味着我们可以提供一个特性，在常规则的消息队列中无法提供的特性。比如，消息读取后，不是立刻被删除，我们会保存一段时间再删除。

## 4.3 Efficiency

我们花了大量的力气来提高效率。我们主要的一个案例是处理网站的活动数据，具有很大的容量。每个页面都会有很多的读写。此外，我们假定每条发布的消息可以至少被一个消息者读取，因为我们假定数据消费的代价足够小。

我们也发现，根据经验来看，建立运行许多类似的系统，高效的多租户操作，效率是关键；下游的基础设施很容易会变成瓶颈由于在使用应用时生成时突然生成大量数据；我们可以确保应用可以在负载之下。这是非常重要的，当建立一个中心化的服务向很多应用提供服务，使用方式每天都会发生变化。

我们在之前的章节讨论过磁盘性能。一旦磁盘访问问题被排除，还有两种常见的导致系统低效的情况：很多小的i/o操作，以及大量的字节拷贝；

小I/O问题会发生在客户端以及server端，以及server持久化操作上。

为了避免这个问题，协议上把一组消息定义为一个消息集合"message set"。这允许网络批量请求数据而不是一次只发一条消息。这样的话，Server可以一次性批量保存消息到log文件中，consumer也可以一次获取更多（linear chunks）的数据；
这个简单的优化，导致性能提高很大。批量导致更多的网络包，更多的顺序磁盘读写，持续的内存块等。这也使得kafka把突发的消息随机写变成了顺序写。

另一个低效操作是字节拷贝。消息少的情况下，这不是一个问题，但是当负载比较高的情况下，影响很大。为了避免这个问题，我们定义了标准的消息格式（生产者,broker，consumer）

日志文件是一个目录下的多个文件，每个文件都使用相同的格式（生产者和消息者使用的）将一系列的消息内容与入到磁盘。使用共同的格式使得可以优化很多重要的操作：网络传输以及批量日志文件持久化。现代操作系统使用setfile来优化实现直接将数据发送到socket，而不经过pagecache.

常见的将数据从文件传输到socket，有以下4个步骤：
- 操作系统从磁盘读取数据到pagecache
- 应用从pagecaceh读取用户缓存
- 应用写数据到pagecaceh和socket缓存
- 操作系统从socket缓存中复制数据到nic缓存，然后发到网络；

这明显很低效，有4次拷贝，两次系统调用；使用sendfile，可以实现数据从pagecache直接到network。所以这次优化中，只有最后的拷贝是必须的。

使用零拷贝技术，数据直接从pagecache中拷贝一次，然后在每次消费时被重复使用，而不是数据存储在内存中，每次读时再拷贝到用户空间。这可以实现数据的消息接近网络带宽。

### 端到端批量压缩

在某些场景下，瓶颈不是CPU或者磁盘，而是带宽。特别是消息传输需要经过广域网的时候。
kafak支持以一种高效批量的格式压缩；一个批量的消息可以压缩后发给server。这个批量的消息会以压缩的方式写入log，然后只有consumer可以解压。

kafka支持GZIP, Snappy and LZ4 压缩算法。


## 4.4 The Producer

### 负载均衡

producer会直接把数据发送到分区的leader所在的broker，而不需要任何的路由。为了实现这个功能，kafka的每个结点都需要能够及时响应请求关于元数据信息，如哪些server是可用的，分区的leader在哪台机器上，以便于producer把请求转发。

producer可以控制把消息发送到哪个分区上。这可以基于随机的负载均衡，或者以某种分区算法。我们提供了分区算法的接口，允许用户自定义实现。这些允许consumer消费他们感兴趣的数据。

### 异步发送

批量发送是高性能的一个主要的因素，producer使用批量发送会计算内存中的数据并且在一个请求中发送多个批量数据。批量发送可以配置在等待一定数据量以及等待一定时间后发送。这允许累积发送更多的数据，减少大量的I/O操作。这种方式需要增加一定的延迟来达到更好的吞吐。

## 4.5 The Consumer

consumer通过发送“fetch”请求到分区为leader的broker，消费他们想消费的数据；consumer会在每个请求中指定消息在log中的偏移，然后会收到从那个位置起的一批消息。因此consumer对消息位置有足够的控制权，并且如果有需要的话，可以重新消费数据。

### Push vs. pull

我们考虑的首要问题是consumerr抽取数据，还是broker推送数据。像许多消息系统一样，kafka遵循了这种设计，producer推送数据到broker，consumer从broker抽取数据。一些日志中心化的系统，如scribe,apache flume，采用的是推送的策略，数据会推送到下游。这种方案各有利弊。然而由于服务端控制数据传输速率，推送系统很难为不同的消费者制定不同的策略。虽然目标是为了消费者可以尽可能快的消费，但是推送系统可能会导致消费者过载，当消费速率慢于生产速率的时候（例如拒绝式攻击）。抽取模式有更好的特点是consumer可以进行速率的控制，速度变慢也可以变快。这也能够减轻一些协议的缺点，consumer已经过载了，无法充分利用传输速率。之前使用这种方案建立系统，有过一些尝试，因此我们采用了这种抽取模型。

由消费者拉模型的另一个好处是可以让consumer自己拉取批量数据，批量自己控制。推送系统一定要选择要么立刻发送一个请求，要么积累很多的数据后，在不了解下游consumer是否具有立刻处理这些数据能力的情况下，再发送到下游。为了降低延迟，也可以不使用缓存数据，而是一次只发送单条数据，这种情况下比较浪费。抽取模型解决了这些问题，consumer可以控制抽取所有可用的消息。因此可以取得优化的批量，而不是引入不必要的延迟。

简单的抽取模型的不足是：如果broker没有数据时，会导致consumer频繁轮询等待数据到达。为了解决这个问题，我们引入参数，在一个请求中，consumer会进行阻塞待一定的时间，直到有数据；（或者等待一定量的数据）

你可以想到的其他一些可能的设计如pull和end-to-end。producer也入本地log文件，然后当consumer抽取数据时，broker再从producer进行抽取。一个类似的“存储转发”producer经常被推荐。这个方案很有趣，但是我们认为对于我们的设计场景案例并不太适合。

根据我们的经验，大规模运行持久化的数据系统，会涉及很多应用数千个磁盘，这些不会使得系统更可靠，甚至可能是个恶梦。实际上，我们发现我们可以通过大规模使用SLA（SLA：Service-Level Agreement的缩写，意思是服务等级协议。）的流管道，而不需要producer的持久化。

### Consumer Position（消费者的位置）

跟踪哪些消息被消费，是消息系统高性能的关键点之一。

大多数消息系统都会保留元数据信息，关于哪些消息已经被消费过了。也就是说，当消息发给consumer后，broker会立刻记录或者等待consumer的一个ack。这是一个直观的选择，也是一个务实的选择，因为只有broker知道哪些消息被消费，哪些可以删除，来保证数据的空间。

然而让broker和consumer都认可关于什么是被消费，是一个不太容易被发现的大问题。如果broker每次发出消息后，就认为消息被消费，但是consumer没有成功处理消息（如宕机或者处理超时），那么消息会丢失。为了解决这个问题，许多消息系统增加了响应机制，broker会等一个consumer返回一个特定的响应来表明消息被消费了。这个方案解决了消息丢失的问题，但是也产生了其他问题。首先：如果消息者处理完消息后发送ack失败，会导致消息被消费两次。第二个是关于性能，broker会跟踪每个消息的状态。

kafka以另外的方式进行处理。topic被分成很多有序的分区，每个分区只能一次被一个group中的消费者消费。这意味着一个消费者在一个分区中的位置只是一个数字，下一条消息要被消费的偏移。这使得保存这些状态代价比较小。这些状态可以定期写入检查点。

这种设计产生了另外一个好处。一个消息者可以重新消费数据。这违反了一个常见的队列的定义，但是对于很多的consumer来说是非常必要的。例如：如果consumer在消费数据后，发面代码有bug，在bug处理好后，consumer可以重新消费数据。

### 离线数据加载

可扩展的持久化存储允许consumer定期消费数据，把数据批量加载到离线系统如：hadoop或者关系数据库；
hadoop场景下，可以通过分区实现任务的拆分，并行执行。

## 4.6 消息传递机制

有多种消息传递机制，保证消息被传递
* 至多一次：消息可能会丢失，但是不会重复传输
* 至少一次：消息不会丢失，但是可能会重复传输
* 精确一次：大家需要的，每条消息只会被传输一次

这会产生两个问题：发送消息的保证 以及 消费消息的保证的持久性；

很多系统都声称提供“精确一次的”传输机制，但是大多声明都存在误导。（他们不会说明当有consumer或者producer失败的情况，或者有很多consumer进程，或者磁盘数据丢失的情况下）

kafka的机制是很直接的。当正在发布一条消息的时候，我们会有一个消息正在提交的说明。一但一条发布的消息被提交完成，只要相应broker上相应的分区副本是存活的，那么数据就不会丢失。提交后消息的定义，存活的分区，以及我们会尝试修复处理哪些异常失败情况会在下个章节进行说明。现在我们假定一个完美，不会丢失数据的broker，并且了解这种对于producer和consumer的保障机制。如果一个producer尝试发布一条消息，并且刚好发生了网络异常，那么它无法保证是否在消息提交前或者提交后会发生故障；算法类似插入一个数据库中的自动生成主键的表。

在0.11.0.0之前的版本，当一个producer没有收到一个说明消息被提交完成的响应；那么基本说明提交成功的概率很低，最好重发消息。这提供了至少一次的传递机制，因为最初的事实上很有可能是成功的，所以重发消息会导致消息又被提交到日志中。从0.11.0.0开始，producer也开始支持幂等消息传递机制，来保证重发的消息不会在日志系统中重复。为了实现这个特性，broker会分配每个producer一个ID，并且每一条由produer发送的消息也会有一个序列。从0.11.0.0开始，producer也支持发送消息到多个topic分区，使用事务特性，例如：所有的消息会部被成功写入，或者全部失败。这个主要的用例场景是精确一次的处理多个topic。

并不是所有的用例都需要如此强的机制特性。如果用户对于延迟要求很高，那么我们可以配置producer参数来达到要求。例如producr可以指定上一条消息成功提交后，再执行后续操作，配置10ms的等待时间。 当然，producer也可以指定以异步方式发送消息，或者只需等待到leader成功接受消息即可。

现在我们以consumer的角度来描述这些特性。所有的副本都有完全一致的日志log，和一致的offset。consumer控制读取消息在log中的位置。如果consumer进程从来不会宕掉的话，它可以在内存中保存这个位置，但是当consumer进程宕掉后，我们希望其他consumer进程来继续消费这个分区的时候，这个进程需要有一个合适的位置来进行处理。它有几个方案来取得消息，更新位置。

- 读取消息，保存当前读取位置，再处理消息。这种情况下，有一种可能是消费者保存位置后，但是保存处理消息输出之前，进程宕掉了。这种情况下，接管消息处理的进程可以从保存位置开始进行消息处理，但是可能有一些消息在offset之前没有被处理。这种情况对应了至多一次的消息机制，存在consumer进程宕掉，部分消息没有处理。
- 读取消息，处理消息，再保存当前消息位置。这种情况下，有可能consumer进程在处理完消息后，在保存当前消息位置之前，进程宕掉了。这种情况下，新的进程会重新读取到之前已经处理过的消息。这对应consumer进程异常时，消息至少传递一次的机制。在许多的场景下，消息都会有一个主键，因此更新可以是幂等的（收到两次同样的消息，但是只更新一次数据库）

那么精确一次的机制是什么？（事实上是你真正想要的）当从一个kafka topic中消费数据，并保存到另外一个topic（类似kafka streaming应用），我们可以使用在0.11.0.0中的producer事务特性。消费者的消息位置是被当作消息保存在topic的，所以我们当我们向topic中写入数据的时候，也可以在同一个事务中写入offset。如果事务中断，那么consumer的读取位置也会被回退到旧的值，而且基于隔离级别，目标topic中的内容可以不对其他的consumer可见。默认是“脏读（read_uncommitted）”级别，所有的消息对consumer都是可见的，即使事务失败了，但是read_committed，消费者只会返回事务成功的消息（以及非事务的消息）。

当消息写入外部系统时，限制是需要协调consumer的消息位置以及真正保存的数据。传统的方式是在保存consumer位置以及consumer输出之间引入两阶段提交协议。这个可以更加简单的处理为：consumer在同一个地方保存offset和处理结果。这是一个更好的方案，因为很多consumer需要写入的系统不支持两阶段提交协议。举个例子，可以考虑使用kafka connect将数据以及消息的offset保存到HDFS上，以便于保证数据和offset要么全部提交，或者全部失败。我们的很多数据系统，需要很强的一致性机制来保证，而且消息没有主键无法实现去重，因此都参考类似的方案。

所以kafka可以有效的支持精确一次的传输机制在kafka-streams，而且在kafka topic之间传输以及处理数据时，可以使用事务的producer/consumer可以实现精确一次传输。其他目标系统使用精确一次传输机制需要与其他系统协作，但是kafka提供的Offset可以使得这种协作更加灵活。其他方面，kafka保证了至少一次传输，并且允许用户实现至多一次传输，通过禁止producer重试和consumer在批量处理消息前提交Offset。

# 4.7 副本
kafka会在已配置过的一定数量的机器间，复制每个topic分区的日志文件（可以配置副本数量）。这实现了自动的故障切换，当集群中有一台服务器宕机时，消息仍然是可用的。

其他的消息队列也提供了一些副本相关的特性，但是以我们的观点来看，其实有许多缺点，从结点不是激活状态，吞吐被严重影响，而且需要手工切换等。kafka是被设计默认使用副本的，事实上我们也可以通过把副本数设为1，实现没有副本的topic。

topic的分区是副本的最小单位。在没有故障的情况下，在kafka下的每个分区都有一个leader和零个或者多个follower。副本总数包含leader组成了副本因子。所有的读写都会通过分区leader来实现。例如：有许多超过broker的分区，并且leader会最终分布在这些broker中。这些在followers的消息数据是和leader中的数据一致的，所有都有一样的offset，以及消息有相同的顺序。（当然，在某种特定时间下，leader也会有许多未同步的消息在log文件的末尾）

Follower像普通的一个kafka consumer一样，从leader消费数据，并且添加到自己的log文件中。

对于大多数可以自动进行故障处理的分布式系统，需要精确定义一个结点存活的标准。对于kafka 有两个条件：
- 一个结点一定要可以保持zookeeper的会话（通过zookeeper的心跳机制）
- 如果是从节点（slave节点），那么它必须在leader写后立刻同步数据，而且不能落后leader太多

我们定义了满足以上两个条件的结点才被认为是同步状态的，避免了存活以及失败两种模糊状态；leader会一直跟踪同步状态的结点。如果一个follower进程宕掉了或者僵死，或者无法和Leader保持一致的，相差太远，那么leader会把它从同步副本结点列表中删除掉。可以通过配置参数` replica.lag.time.max.ms`来判断副本是否僵死或者延迟太多。

在分布式系统中我们只会尝试处理故障恢复模型下的故障，结点会突然停止运行然后后续恢复。kafka不会处理所谓复杂的故障，结点会产生随意或者恶意的响应。

我们精确定义了当所有同步的副本结点都已经把消息添加到自己的日志文件中，才认为这条消息提交完成；只有提交的消息才能提供给consumer。这意味着消息可能会丢失如果leader发生故障。另一方面，Producer可以一直等待消息提交完成也可以不等待，这取决于他们对于延迟和可靠性的取舍，可以通过Produer的参数acks来进行控制。topic会配置一个最小同步副本数，当producer等待副本提交消息完成响应时，会检查这个参数。如果producer只请求一个比较小的ack参数，那么即使当前同步副本数小于最小同步副本数，那么这条消息会被认为提交完成以及消费完成。

这保证了kafka不会丢失提交完成的消息，只有有一个同步的结点是存活的。

结点宕机后，会经过很短的切换时间，kafka仍然可用；但是如果发生网络分裂（脑裂，network partition），则kafka不会可用；

### 副本日志: 仲裁, 同步结点, and 状态机 (Oh my!)

一个kafka分区的核心是一个副本日志；在分布式系统中，副本日志是最基本的一个概念，而且有许多方案实现。一个副本日志可以以状态机的方案来实现（state-machine style）。

副本日志会顺序处理一系列的数据。有许多多的方式实现这个功能，但是最简单快速的方式是通过一个leader来定义数据的顺序。只要leader是存活的，所有的follower需要复制这些数值，并且以leader的顺序。

当然如果leader不会宕掉，我们不需要follower。当leader宕掉后，我们需要从follower中选择一个新的leader。但是follower可能会有落后于leader，所以我们必须选择一个最接近leader的follower。日志同步算法必须保证如果我们告诉client一个消息已被提交完成，如果leader宕机了，那么新选择的leader一定要有那条消息。这就会产生一个权衡：如果leader等到足够多的follower反馈消再声明消息提交成功，那么将会有更多潜在可选的leader。

如果选择一定数量的响应以及一定数量的日志来比较选择一个leader，以便于保证有重叠，这就叫做法定数；（Quorum）


一个普遍的方案来权衡是对于提交的判断以及leader的选举通过使用绝大多数投票来实现。这不是kafka使用的，但是我们可以来了解这种权衡。例如我们有2f+1个副本，如果f+1个副本一定要收到消息，leader才能声明消息已提交完成。通过选择从f+1副本且包含最完整日志的follower中 ，我们可以选择一个新的leader，只要不超过f个宕掉，leader可以保证拥有所有提交的消息。因为在f+1个副本中，至少有一个副本包含所有的消息。副本日志会是最完整的，因此可以被选择做为leader。有许多的细节算法需要处理（例如：精确定义日志有多完整，保证日志一致性当leader失败或者修改副本Server集），我们现在忽略这些。

大多数投票算法有一个特点：延迟是依赖于最快的Server。这意味着，如果副本数是3，延迟是由最快的slave决定的，而不是最慢的。

有许多类似的算法，如ZooKeeper's Zab, Raft, and Viewstamped Replication. kafka使用的是Microsoft的 PacificA 。

多数投票算法的缺点是有多太的结点失败会导致无法选择leader。容忍一个失败需要三个副本，容忍两个失败需要五个副本。以我们的经验来看需要足够的冗余来容忍一个单一的失败是不够的对于一个实际的系统，每次都写五次，需要5倍的磁盘空间以及1/5的吞吐，这是一个数据容量问题，并不实用。这也是为什么多数法人算法更普遍出现于共享集群配置例如Zookeeper，但是很少出现在数据存储系统。例如：HDFS的namenode是基于多数投票journal算法来实现高可用，但是这种高代价的方案并没有用在数据存储本身上。

kafka选择了有一点点不同的算法来选择投票法人集合。kafka动态维护了一个同步副本集合来替代多数投票。只有这个集合中的成员才可以被选择成为leader。直到所有的同步副本收到写操作，那么写入kafka分区才被认为是提交完成。当ISR集合变化时都会被持久化到zookeeper中。因为这个原因，ISR中的任何一个副本都可以被选择为leader。这对于kafka使用模型来说是一个非常重要的因素，来保证leader平衡。通过ISR模型以及f+1副本，一个kafka的topic可以容忍f个失败而不丢数据。

我们希望处理的大多数情况是，我们希望这种权衡是合理的。实际上，为了容忍f个失败，提交消息时，大多数投票以及ISR方法都会等待相同副本响应。（例如，为了容忍一个副本失败多数投票算法需要等3个副本和1个响应，ISR算法需要2个副本和1个响应）。多数投票算法的优势是提交消息时不需要等待最慢的Server。然而，我们认为可以通过客户端来选择是否阻塞消息的提交，或者额外的吞吐，以及磁盘的空间，由于较少的副本数值得它。

另外一个重要的设计是kafka不需要故障结点进行全部数据的恢复。`这是普遍的对于副本算法依赖稳定存储的存在，不能在任何的故障恢复场景下丢失数据，不能有任何潜在的不一致的情况。`这种假定下有有两种主要的问题。第一，磁盘问题是我们在实际操作和持久化的数据系统中观察到的最普遍的问题，他们经常使得数据无法保持完好无损。第二、即使这不是问题，我们也不需要每次写后都要进行磁盘同步来实现一致性保证，因为这会减少两到三个数量级的性能。我们的协议允许一个副本重新加入ISR，它需要再次完全同步，即使它丢失了未刷新的数据当它宕掉的时候。

### unclean leader 选举：万一所有结点都挂了？

kafka能够通过至少一个副本是存活保证数据不丢失；如果分区的副本进程都死掉了，那这个保证是不存在的；

然而，当所有的副本进程都死了的时候，一个具体的系统需要做一些合理的操作；当你不幸遇到这种情况的时候，分析原因是非常重要的。有两种操作：

- 等ISR的副本进程重新启动，并且把这个副本选为leader（希望数据是完整的）
- 选择第一个启动的副本作为leader（不一定是在ISR中）

这是在可用性和一致性上做的权衡。如果我们等ISR的副本启动，那么只要副本没有启动，kafka会一直不可用。如果副本损坏或者数据丢失，那么我们可能会永远无法启动。另外一方面，如果我们允许一个非同步的副本启动后选为leader，那么即使无法保证每条提交后消息的可用性，但是可以做为一个方案。默认情况下，kafka会选择第二种方案，当所有ISR挂掉后，会选举一个不一致的副本做为leader。这个功能可以通过参数来禁用，在一些用户情况下，一致性更加重要。

这种进退两难的情况并不只是针对kafka。这种情况出现在很多基于选举算法的scheme中。例如，在多数投票算法，如果大多数server都永久性失败，那么你要么选择100%丢失数据，要么丢失一致性，选择存活的server做为你新的数据源。

### 可用性和持久性保证

当写入kafka的时候，producer可以选择等待副本响应的个数（0，1，全部）。注意所有副本的响应并不保证全部分配的副本收到了消息。默认情况下，acks=all，表示只有目前所有同步的副本收到消息。例如：如果一个topic配置为只有两个副本，并且一个挂了（例如只有一个同步副本），然后写入会成功。然而，如果存活的副本也挂了，那么这些写入会丢失。尽管这保证了分区的最大可用性，但是可能不是一些用户想要的，他们更需要的是持久性，而不是可用性。所以我们提供了两种topic级别的配置，来选择持久和可用生。

- 禁止“非同步副本”选举：如果所有的副本变得不可用，那么只有当leader可用时，分区才会可用。这有效的减少了消息的丢失，但是导致不可用。
- 指定最小的ISR大小-为了减少写入一个副本时突然不可用，导致丢失数据的情况，分区只接受入超过指定最小值的ISR大小。这个配置只有在producer使用了ack=all，并且保证消息至少会被这些同步副本回应时才生效。这个设置在一致性和可用性间提供了权衡。最小ISR数配置过高可以保证更高的一致性由于消息会被保证写入到更多的副本中，减少了数据丢失的可能性。然而减少了可用性，当同步副本数低于最小阈值的时候，分区会变得不可用。

### 副本管理

以上关于副本的讨论只是关于一个topic的分区副本的讨论。然而kafka集群会管理数以千计的分区。一个集群内，我们会尽力以轮询的方式分散数据，避免大容量的分区分布一小部分机器上。就像我们会尽力平衡leadership以便于保证每个结点都有合适数量分区的leader。

由于leader的选举过程会导致不可用，因此优化leader的选举过程也是非常重要的，A naive implementation of leader election would end up running an election per partition for all partitions a node hosted when that node failed. 相反，我们选择一个brokeer作为controller。controller检查服务异常，并且会对发生异常的broker所影响的分区更换leader。这个结果是我们可以批量发布更换leader的通知，这使得当进行大量的分区leader时，选举过程更加的快速代价更小。contoller挂了后，会从存活的broker中选择出新的controller。

# 4.8	日志压缩


# 4.9 限额

kafka集群






























