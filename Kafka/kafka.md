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

### Topic && Partition

​	为了使`Kafka`的吞吐量线性提高，物理上把`Topic`分成一个或者多个`Partition`，每个`Partition`在物理上对应一个文件夹，文件夹下面存储这个`partition`所有的消息和索引文件。

每条消息都有一个当前 Partition 下唯一的 64 字节的 offset，它指明了这条消息的起始位置。磁盘上存储的消息格式如下：

```
message length ： 4 bytes (value: 1+4+n)
"magic" value ： 1 byte 
crc ： 4 bytes 
payload ： n bytes 
```

### Segment



## Kafka的持久化





