

##  Kafka Producer

### 负载均衡

Kafka API暴露出方法可以指定`Producer`发送的消息要到达具体的`Partition`。可以通过实现接口中具体的方法来进行。`Partition`中的Message是有序的，但是整个Topic中的消息并不保证有序 。

### 异步发送

批处理，Producer 在向broker发送数据时，可以通过异步批处理的方式进行，这样可以减少IO的开销。

## Kafka delivery guarantee

1. At most once.
2. At least one 
3. Exactly once 

kafka默认支持的是 至少消费一次。

#### Producer 端

生产者push消息，会收到Kafka broker 响应是否成功的标示，如果中间发生网络故障，Producer 没有收到Kafka broker 响应的成功标示，那么它会重发这个消息。至于Broker,它会为这个重发的消息做一个幂等校验，broker为每个 Producer分配一个ID，并且通过Producer 发送来的每个消息的`sequence number` 作为幂等校验，并且这个`Sequence Number`会进行持久化，所以，及时`leader of broker`宕机，新的`leader`被选举出以后依然可以持续维护这个`Sequence Number`。通过这种机制在Producer端实现最多一次也很简单，只send一次，不进行重试即可。



#### Consumer 端

在Consumer端，每次Consumer在消费了pull到消息且consumer以后要 commit 到broker 在topic 维护的metadata中自增其消费的offset，如果消费了没有自增，那么就会造成重复消费。

以上的At most once的场景是在 Kafka高可用的前提下才能实现。

总之，Kafka 默认保证 At least once，并且允许通过设置 Producer 异步提交来实现 At most once。而 Exactly once 要求与外部存储系统协作，幸运的是 Kafka 提供的 offset 可以非常直接非常容易得使用这种方式。

#### 怎么实现准确的一次消费

原子提交（atomic commit）

对于Producer而言，如果Kafka端做了幂等完全可以实现消息的一次push，但是对于`Producer`向多个Topic进行消息的push，`Producer` 可以利用 `batch send` 和两部提交做到atomic的实现,如下：

```java
producer.initTransactions();
try {
  producer.beginTransaction();
  producer.send(record1);
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

在Consumer端，通过`Kafka`人配置的隔离级别`read_uncommit` 和`read_commit`两种隔离级别。



### Replication

