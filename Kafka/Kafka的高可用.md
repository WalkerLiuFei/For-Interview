# Kafka 高可用

## Data Replication

## Leader Election

## Kafka HA设计解析

为了更好的做负载均衡，Kafka 尽量将所有的 Partition 均匀分配到整个集群上。一个典型的部署方式是一个 Topic 的 Partition 数量大于 Broker 的数量。同时为了提高 Kafka 的容错能力，也需要将同一个 Partition 的 Replica 尽量分散到不同的机器。

Kafka 分配 Partition和 Replica 的算法如下：

1. 将所有 Broker（假设共 n 个 Broker）和待分配的 Partition 排序
2. 将第 i 个 Partition 分配到第（i mod n）个 Broker 上
3. 将第 i 个 Partition 的第 j 个 Replica 分配到第（(i + j) mode n）个 Broker 上

Kafka 的 Data Replication 需要解决如下问题：

- 怎样 Propagate 消息 ：
- 在向 Producer 发送 ACK 前需要保证有多少个 Replica 已经收到该消息 ： 
- 怎样处理某个 Replica 不工作的情况
- 怎样处理 Failed Replica 恢复回来的情况

 Produer只向Partition的leader发送消息，leader将消息写入到本地的Log，然后每个Partition的Follower都向Leader Pull消息，完成之后（Follower持久化之前）向Leader发送ACK消息。

那么什么时候能够通知**Producer** 这条message被消费了哪？考虑两种方式：

1. 所有的Follower将Leader中的消息同步到内存中，这样可以防止很大概率的防止数据丢失，并且更加高可用。即当leader 宕机以后可以由更多的Follower可以作为leader的候选。缺点是这样很大程度上限制了Kafka的吞吐率。
2. 放Leader中的消息只要被持久化，即可响应给`Producer` ACK，因为持久化以后可以保证消息不丢失。如果以这种，其他Follower还没有进行同步就响应ACK的方式，很有可能会造成数据的一致性问题。

`Kafka`选择了一种折中的方式解决这种问题，它定义了一个最小的 replica-number设为f，当达到这个f个Follower同步leader中的最新message并且响应给leader ACK以后，那么leader就会将对Producer响应ACK。当Leader宕机时，只有那些同步了最新message的Follower （称为ISR 成员）有资格被推选为Leader。通过这种方式达到 **数据一致性和高吞吐率的一个平衡**。

如果出现这种问题，ISR中所有Follower和Leader 全部宕机怎么办，这个时候Kafka有两个选择（可以进行配置）

1. 保证数据一致性 ，放弃可用性： 等待ISR中的Follower 活过来。
2. 保证可用性，放弃一致性 ： 选择第一个活过来(或者活着的Follower)作为Leader。

### Leader的选取

Kafka集群中所有的节点管理都是由`Zookeeper`完成的。包括leader选取等等。

Kafka 0.8.* 的 Leader Election 方案解决了上述问题，它在所有 broker 中选出一个 controller，所有 Partition 的 Leader 选举都由 controller 决定。controller 会将 Leader 的改变直接通过 RPC 的方式（比 ZooKeeper Queue 的方式更高效）通知需为为此作为响应的 Broker。同时 controller 也负责增删 Topic 以及 Replica 的重新分配。

partition state（`/brokers/topics/[topic]/partitions/[partitionId]/state`） 结构如下：

```json
Schema:
{ "fields":
    [ {"name": "version", "type": "int", "doc": "version id"},
      {"name": "isr",
       "type": {"type": "array",
                "items": "int",
                "doc": "an array of the id of replicas in isr"}
      },
      {"name": "leader", "type": "int", "doc": "id of the leader replica"},
      {"name": "controller_epoch", "type": "int", "doc": "epoch of the controller that last updated the leader and isr info"},
      {"name": "leader_epoch", "type": "int", "doc": "epoch of the leader"}
    ]
}

Example:
{
    "controller_epoch":29,
    "leader":2,
    "version":1,
    "leader_epoch":48,
    "isr":[2]
}
```

`/controller -> int (broker id of the controller)`存储当前 controller 的信息

```
Schema:
{ "fields":
    [ {"name": "version", "type": "int", "doc": "version id"},
      {"name": "brokerid", "type": "int", "doc": "broker id of the controller"}
    ]
}
Example:
{
    "version":1,
　　"brokerid":8
}
```

### broker failover 过程简介

1. Controller 在 ZooKeeper 注册 Watch，一旦有 Broker 宕机（这是用宕机代表任何让系统认为其 die 的情景，包括但不限于机器断电，网络不可用，GC 导致的 Stop The World，进程 crash 等），其在 ZooKeeper 对应的 znode 会自动被删除，ZooKeeper 会 fire Controller 注册的 watch，Controller 读取最新的幸存的 Broker。

2. Controller 决定 `set_p`，该集合包含了宕机的所有 Broker 上的所有 Partition。

3. 对 set_p 中的每一个 Partition 

   3.1 从`/brokers/topics/[topic]/partitions/[partition]/state`读取该 Partition 当前的 ISR   

   3.2 决定该 Partition 的新 Leader。如果当前 ISR 中有至少一个 Replica 还幸存，则选择其中一个作为新 Leader，新的 ISR 则包含当前 ISR 中所有幸存的 Replica。否则选择该 Partition 中任意一个幸存的 Replica 作为新的 Leader 以及 ISR（该场景下可能会有潜在的数据丢失）。如果该 Partition 的所有 Replica 都宕机了，则将新的 Leader 设置为 -1。   

   3.3 将新的 Leader，ISR 和新的`leader_epoch`及`controller_epoch`写入`/brokers/topics/[topic]/partitions/[partition]/state`。注意，该操作只有其 version 在 3.1 至 3.3 的过程中无变化时才会执行，否则跳转到 3.1

4. 直接通过 RPC 向 set_p 相关的 Broker 发送 LeaderAndISRRequest 命令。Controller 可以在一个 RPC 操作中发送多个命令从而提高效率。 

   broker failover 顺序图如下所示。   

   ![img](https://static001.infoq.cn/resource/image/1e/2d/1e39d5d6165cf0117b537589a660e82d.png)

### 创建 / 删除Topic

1.  ` Controller` 在 ZooKeeper 的`/brokers/topics`节点上注册 Watch，一旦某个 Topic 被创建或删除，则  `Controller` 会通过 Watch 得到新创建 / 删除的 Topic 的 Partition/Replica 分配。
2. 对于删除 Topic 操作，Topic 工具会将该 Topic 名字存于`/admin/delete_topics`。若`delete.topic.enable`为 true，则 Controller 注册在`/admin/delete_topics`上的 Watch 被 fire，Controller 通过回调向对应的 Broker 发送 StopReplicaRequest，若为 false 则 Controller 不会在`/admin/delete_topics`上注册 Watch，也就不会对该事件作出反应。
3. 对于创建 Topic 操作，Controller 从`/brokers/ids`
   1.   从分配给该 Partition 的所有 Replica（称为 AR）中任选一个可用的 Broker 作为新的 Leader，并将 AR 设置为新的 ISR（因为该 Topic 是新创建的，所以 AR 中所有的 Replica 都没有数据，可认为它们都是同步的，也即都在 ISR 中，任意一个 Replica 都可作为 Leader）
   2. 将新的 Leader 和 ISR 写入`/brokers/topics/[topic]/partitions/[partition]`

4. 直接通过 RPC 向相关的 Broker 发送 LeaderAndISRRequest。 

   <img src="https://static001.infoq.cn/resource/image/51/d6/51d051dfffd50ca924f11a51233274d6.png" alt="img" style="zoom: 50%;" /> 



