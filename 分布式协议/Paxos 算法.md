#  Paxos 算法

[参考](https://www.zhihu.com/question/19787937) 

[参考，youtube](https://www.youtube.com/watch?v=d7nAGI_NZPk)

Paxos的算法是一个最终一致性的算法，



## Paxos 基本概念

Paxos have three roles , `proposers`,`acceptors` ,`learners`，paxos 节点可以同时扮演三个角色。

Paxos 节点必须知道acceptors的多数是什么

Paxos 节点必须要对数据进行持久化，Paxos不能将接受到的数据遗忘掉。 	

Paxos 的目标是达到一个唯一的最终一致性，一旦一个一致性达成，是无法回滚的



## Paxos 一致性达成过程

1. Proposer 选择一个 `Prepare IDp` （此ID 必须唯一）向所有的的`acceptor`,如果本身也是`acceptor`，那么本身也是需要发送到。

2. `acceptor` 如果同意`Proposer`的发起，那么`acceptor`会忽略其他`proposer  ` 发来的低于此`Prepare ID `的`Prepared ID`值。  然后`acceptor` 会将 `promise IDp`响应给`Proposer`
3. `Proposer` 得到了`IDp`足够多（大多数）的回复，那么`proposer` 会向大多数的节点发送`Accept-request IDp, value`  消息。
4. `Acceptor`收到了`IDp`的`Accept-REQUEST ` value, 如果`acceptor` 同意这个请求，`acceptor`接收这个`REQUST` ,并响应给`Proposer` ，`ACCEPT IDp`的消息。并将`ACCEPT IDp`的消息同步给所有的leaner.
5. 当`Proposer` ，`leaner` 收到了`ACCEPT` 消息，那么全部的流程结束。`IDp`的消息就是value. 



## 两阶段提交



## 两阶段提交，三阶段提交，Raft,Paxos的区别

[raft 算法概述](https://zhuanlan.zhihu.com/p/32052223)

[2 phase commit vs paxos](https://stackoverflow.com/questions/27304887/paxos-vs-two-phase-commit)

Raft 算法是工程化的`Paxos`一致性算法，由于`Paxos` 算法太过于工程化。