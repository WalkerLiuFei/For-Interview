## Kafka实现唯一消费的方式

1. [Kafka 实现唯一消费的方式](https://www.infoq.cn/article/kafka-analysis-part-8)
2. [transaction in kafka](https://www.confluent.io/blog/transactions-apache-kafka/)
3. [exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/) 

要解决的问题

1. 生产端的重复发送
2. 消费端的重复消费
3. 多重实例消费，或者多重实例发送消息的问题

### 一个现实场景

1. 进程P0 向topic `tp0` 写入一个消息A
2. 进程P1通过`tp0`读取消息A，并生成消息B = F(A)
3. 进程P1向topic `tp1`写入消息B
4. 进程P0通过`tp1`消费消息B

如果要做到只消费1次，这个场景需要解决的问题 ：

+ P0不能重复向`tp0`写入消息A**（通过消息的幂等性解决）**
+ 进程P1 不能重复消费 消息A，且不能重复提交消息B**（通过事务解决）**
+ P0不能重复消费消息B 

### Producer 端

**消息幂等解决生产端的重复发送**

对于Producer而言，如果Kafka端做了幂等完全可以实现消息的一次push，但是对于`Producer`向多个Topic进行消息的push，`Producer` 可以利用 `batch send` 和两部提交做到atomic的实现,如下：

```java
producer.initTransactions();
try {
  producer.beginTransaction();
  producer.send(record1);
   // what if record2 is generate by record1 .
  producer.send(record2);
  producer.commitTransaction();
} catch(ProducerFencedException e) {
  producer.close();
} catch(KafkaException e) {
  producer.abortTransaction();
}
```

只有当 commitTransaction 执行成功以后，事务中的所有消息才会被提交到broker，通过这中方式可以做到**原子提交**。

通过对`Producer`提交消息的幂等校验和`原子提交`。`Producer`端对消息其实相当与已经完成了消息只会推送一次的限制。

### Consumer端

在Consumer端，通过`Kafka`人配置的隔离级别`read_uncommit` 和`read_commit`两种隔离级别。在事务中读取完对应的record后进行offset提交。

### 示例

```java
KafkaProducer producer = createKafkaProducer(
  “bootstrap.servers”, “localhost:9092”,
  “transactional.id”, “my-transactional-id”);

//初始化事务
producer.initTransactions();

// 开启一个consumer
KafkaConsumer consumer = createKafkaConsumer(
  “bootstrap.servers”, “localhost:9092”,
  “group.id”, “my-group-id”,
  "isolation.level", "read_committed");
// 坚挺
consumer.subscribe(singleton(“inputTopic”));

while (true) {
  ConsumerRecords records = consumer.poll(Long.MAX_VALUE);
  //开启事务
  producer.beginTransaction();
  for (ConsumerRecord record : records)
    producer.send(producerRecord(“outputTopic”, record));
  //将consumer 的offset发送到offset topic  
  producer.sendOffsetsToTransaction(currentOffsets(consumer), group);  
  // 事务提交
  producer.commitTransaction();
}
```

## transaction 的实现

对于这种分布式一致性问题，目前我们有`Producer` 和`Kafka topic` 两者，如果要协调好两者的一致性，必然需要一个协调者进来。这个时候就是 `Kafka  Transaction Coordinator` 和`两阶段提交`了。

<img src="https://www.confluent.io/wp-content/uploads/transactions-1-1024x693.png" alt="img" style="zoom: 40%;" />

`transaction coordinator` 是一个运行在`broker` 中的模块，`transaction log`也是`kafka`中的一个`topic`

每个transaction 通过`transaction_id` 对应唯一的一个`transaction coordinator`

### 过程

####  Producer 和 coordinator交互

1. Producer 向 coordinator请求一个唯一的 transaction_id ,并获取到一个epoch，具有相同 PID 但 epoch 小于该 epoch 的其它 Producer（如果有）新开启的事务将被拒绝。
2. 当生产者第一次在事务中将数据发送到分区时，首先在协调器中注册该分区。
3. 当应用程序调用`commitTransaction`或`abortTransaction`时，会向协调器发送一个请求以开始`两阶段提交协议`。

####  coordinator and transaction log 间的交互

coordinator 持续维护transaction的执行状态，并将其写到transaction log 中，transaction log也是一个topic，拥有topic的所有性质（持久化，分区等等）。并且，coordinator是 transaction log 唯一的 R/W的组件。

#### Producer 将数据写到目标上的 topic-partition

在与协调器的事务中注册新分区后，生产者正常地将数据发送到实际分区。这与producer.send流完全相同，但需要一些额外的验证以确保生产者没有隔离。

#### coordinator和 topic-partition之间的交互

在Producer 进行transactionCommit或者abort之后，coordinator开始两阶段提交的操作，

1. **prepare commit（准备提交） :** 更新 coordinator 内部维护的 transaction的状态到`prepare_commit`并持久化到transaction_log ，一旦完成，无论如何都保证交易最终完成(即 ： coordinator 进行重试)。
2. **transaction commit（执行提交） ** ： 它将事务提交标记写入作为事务一部分的主题分区。

对于coordinator的高可用问题，因为broker是集群时的，而coordinator是broker的一个组件，所以不用担心起高可用问题。

## 事务过期机制

#### 事务超时

通过配置 ：

```
transaction.timeout.ms
```

#### 终止过期事务

当 Producer 失败时，`Transaction Coordinator`必须能够主动的让某些进行中的事务过期。否则没有 Producer 的参与，`Transaction Coordinator`无法判断这些事务应该如何处理，这会造成：

- 如果这种进行中事务太多，会造成`Transaction Coordinator`需要维护大量的事务状态，大量占用内存
- `Transaction Log`内也会存在大量数据，造成新的`Transaction Coordinator`启动缓慢
- `READ_COMMITTED`的 Consumer 需要缓存大量的消息，造成不必要的内存浪费甚至是 OOM
- 如果多个`Transaction ID`不同的 Producer 交叉写同一个 Partition，当一个 Producer 的事务状态不更新时，`READ_COMMITTED`的 Consumer 为了保证顺序消费而被阻塞

为了避免上述问题，`Transaction Coordinator`会周期性遍历内存中的事务状态 Map，并执行如下操作

- 如果状态是`BEGIN`并且其最后更新时间与当前时间差大于`transaction.remove.expired.transaction.cleanup.interval.ms`（默认值为 1 小时），则主动将其终止：1）未避免原 Producer 临时恢复与当前终止流程冲突，增加该 Producer 对应的 PID 的 epoch，并确保将该更新的信息写入`Transaction Log`；2）以更新后的 epoch 回滚事务，从而使得该事务相关的所有 Broker 都更新其缓存的该 PID 的 epoch 从而拒绝旧 Producer 的写操作
- 如果状态是`PREPARE_COMMIT`，完成后续的 COMMIT 流程————向各`<Topic, Partition>`写入`Transaction Marker`，在`Transaction Log`内写入`COMPLETE_COMMIT`
- 如果状态是`PREPARE_ABORT`，完成后续 ABORT 流程

#### 终止Transaction ID

某`Transaction ID`的 Producer 可能很长时间不再发送数据，`Transaction Coordinator`没必要再保存该`Transaction ID`与`PID`等的映射，否则可能会造成大量的资源浪费。因此需要有一个机制探测不再活跃的`Transaction ID`并将其信息删除。