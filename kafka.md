# Kafka 架构

## 问题

1. Kafka可以以O(1)的时间复杂度进行持久化，即使对 TB 级以上数据也能保证常数时间复杂度的访问性能。

2. 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒 100K 条以上消息的传输 ? 
3. 支持 Kafka Server 间的消息分区，及分布式消费，同时保证每个 Partition 内的消息顺序传输。
   1. 如果消费者和生产者之间
4. 同时支持离线数据处理和实时数据处理。
5. Scale out：支持在线水平扩展。
6. 为什么Redis在处理大数据量的消息时会很慢？
7. Kafka 相对于 `ZeroMQ`，`ActiveMQ`,`RabbitMQ`的区别
8. 使用消息系统的目的
   1. 解耦
   2. 冗余
   3. 扩展性
   4. 灵活性 & 峰值处理能力
   5. 可恢复性
   6. 顺序保证 ： Kafka中的每个partition 可以保证消息是有序的
   7. 缓冲
   8. 异步通信

## Kafka 中的组件

+ Broker : kafka集群中包含一个或多个服务器，这种服务器被称为 broker，相当与路由的角色
+ Topic ：每条发布到 Kafka 集群的消息都有一个类别，这个类别被称为 Topic。
+ Partiition :  物理上的概念，每个Topic包含一个或者多个Partition。
+ Producer : 负责消息发送到Kafka broker 
+ Consumer 
+ Consumer group  ： 每个consumer 属于一个特定的consumer group ，如果未指定，则属于默认的group 

![img](https://static001.infoq.cn/resource/image/bc/c8/bc99c2a3176ee1695a3d4f1f4f08a5c8.png)

Kafka **通过 Zookeeper 管理集群配置**，选举 leader，以及在 Consumer Group 发生变化时进行 rebalance。Producer 使用 push 模式将消息发布到 broker，Consumer 使用 pull 模式从 broker 订阅并消费消息。

### Kafka存储结构

​	为了使`Kafka`的吞吐量线性提高，物理上把`Topic`分成一个或者多个`Partition`，每个`Partition`在物理上对应一个文件夹，文件夹下面存储这个`partition`所有的消息和索引文件(`segment file`)。 每个 `segment file` 又包含 多个 `log entry` 每个entry的数据结构如下：

每条消息都有一个当前 Partition 下唯一的 64 字节的 offset，它指明了这条消息的起始位置。磁盘上存储的消息格式如下：

```c
message length ： 4 bytes (value: 1+4+n)
"magic" value ： 1 byte 
crc ： 4 bytes 
payload ： n bytes 
```

每个 segment 以该 segment 第一条消息的 offset 命名并以“.kafka”为后缀。另外会有一个索引文件，它标明了每个 segment 下包含的 log entry 的 offset 范围。

因为每条消息都被 append 到该 Partition 中，属于顺序写磁盘，因此效率非常高

对于持久化且消费过数据的删除，有两种方式，一是基于时间，二是基于 Partition 文件大小。例如可以通过配置 $KAFKA_HOME/config/server.properties，让 Kafka 删除一周前的数据，也可在 Partition 文件超过 1GB 时删除旧数据，配置如下所示。

**Kafka会为每个每一个`Consumer Group `保留一些metadata信息 即当前消费的Position,也即offset.这个offset由Cousumer控制，Consumer 甚至可以将offset设置为一个 较小的值，对消息进行重复消费，有序offset 是由Consumer 控制，所以broker是无状态的，也就不需要锁机制，这样更加提高了Kafka的消息吞吐率**

### Producer消息路由

### Consumer 

Kafka 保证同一 Consumer Group 中只有一个 Consumer 会消费某条消息，在稳定状态下，每个Partition下的消息只会被某一个特定的`Consumer`消费。也就是说`Kafka` 中消息的分配是按照`Partition`为单位而不是以某条消息为单位的。

+ Group中consumer 数量大于 Partition数量，造成Consumer 会无法消费共同消费的某个Topic的message
+ Group 中consumer数量小于`partition`数量，造成`consumer`会消费多个`partition`的消息，造成消费数据offset不连续。

所以最好的办法就是让 group 下 consumer 和消费的Topic下的Partition 数量一致，这样就可以做到顺序消费。

consumer在Zookeeper上注册Watch完成，每个Consumer 被创建时会触发`Consumer Group`的rebalance

##### rebalance过程

- High Level Consumer 启动时将其 ID 注册到其 Consumer Group 下，在 Zookeeper 上的路径为`/consumers/[consumer group]/ids/[consumer id]`
- 在`/consumers/[consumer group]/ids`上注册 Watch
- 在`/brokers/ids`上注册 Watch
- 如果 Consumer 通过 Topic Filter 创建消息流，则它会同时在`/brokers/topics`上也创建 Watch
- 强制自己在其 Consumer Group 内启动 Rebalance 流程

#### Low Level Consumer

使用 Low Level Consumer (Simple Consumer) 的主要原因是，用户希望比 Consumer Group 更好的控制数据的消费。比如：

- 同一条消息读多次
- 只读取某个 Topic 的部分 Partition
- 管理事务，从而确保每条消息被处理一次，且仅被处理一次

与 Consumer Group 相比，Low Level Consumer 要求用户做大量的额外工作。

- 必须在应用程序中跟踪 offset，从而确定下一条应该消费哪条消息
- 应用程序需要通过程序获知每个 Partition 的 Leader 是谁
- 必须处理 Leader 的变化

使用 Low Level Consumer 的一般流程如下

- 查找到一个“活着”的 Broker，并且找出每个 Partition 的 Leader
- 找出每个 Partition 的 Follower
- 定义好请求，该请求应该能描述应用程序需要哪些数据
- Fetch 数据
- 识别 Leader 的变化，并对之作出必要的响应

#### Consumer Group

使用 `Consumer high level API `时，同一 Topic 的一条消息只能被同一个 `Consumer Group` 内的一个 Consumer 消费，但多个 `Consumer Group` 可同时消费这一消息。这也是实现多播和单播的逻辑，如果实现广播，只需要每个`Consumber` 有一个独立的Group即可。要实现单播只要所有的Consumer在同一个`Group`里面.用 Consumer Group 还可以将 Consumer 进行自由的分组而不需要多次发送消息到不同的 Topic。

<img src="https://static001.infoq.cn/resource/image/68/22/68f2ca117290d8f438610923c108ce22.png" alt="img" style="zoom:50%;" />

### Kafka delivery guarantee

1. At most once.
2. At least one 
3. Exactly once 

kafka默认支持的是 至少消费一次。

生产者push消息，会收到Kafka broker 响应是否成功的标示，如果中间发生网络故障，Producer 没有收到Kafka broker 响应的成功标示，那么它会重发这个消息。至于Broker,它会为这个重发的消息做一个幂等校验，broker为每个 Producer分配一个ID，通过Producer 发送来的每个消息的sequence number 作为幂等校验。

在Producer端实现最多一次也很简单，只send一次，不进行重试即可。

在Consumer端，每次Consumer在消费了pull到消息且consumer以后要 commit 到broker 在topic 维护的metadata中自增其消费的offset（**新版本中，offset是通过 topic进行维护的，为了解决事务的问题**），如果消费了没有自增，那么就会造成重复消费。

以上的At most once的场景是在 Kafka高可用的前提下才能实现。

总之，Kafka 默认保证 At least once，并且允许通过设置 Producer 异步提交来实现 At most once。而 Exactly once 要求与外部存储系统协作，幸运的是 Kafka 提供的 offset 可以非常直接非常容易得使用这种方式。



![Kafkaè®¾è®¡è§£æï¼å­ï¼ï¼Kafkaé«æ§è½å³é®ææ¯è§£æ](https://static001.infoq.cn/resource/image/90/e4/90b3b6149d733a40131201b406f7aae4.png)




