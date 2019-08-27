# Raft 共识算法

## 参考

[Raft 论文](https://raft.github.io/raft.pdf) 

[Consul共识](https://www.consul.io/docs/internals/consensus.html)  
[最好的Raft讲解，没有之一](http://thesecretlivesofdata.com/raft/) 

[参考](https://www.cnblogs.com/mindwind/p/5231986.html) 

## 基本概念

raft 共识算法算是Paxos的工程化，Raft 为了达到共识一共做了两件事情

+ 问题分解
+ 状态简化

问题分解是将"复制集中节点一致性"这个复杂的问题划分为数个可以被独立解释、理解、解决的子问题。在raft，子问题包括，**leader election**， **log replication**，**safety**，**membership changes**。而状态简化更好理解，就是对算法做出一些限制，减少需要考虑的状态数，使得算法更加清晰，更少的不确定性（比如，保证新选举出来的leader会包含所有commited log entry） 

Raft 协议的工作原理

1. 所有节点启动时都是以follower的角色启动的
2. 选举出 leader
3. leader负责replicated log的管理
4. leader  负责所有客户端的请求，然后复制到 follower 节点，并在**安全**的时执行这些操作
5. 如果遇到 leader 故障，followers会重新选举出新的leader



所有的节点在任何时候会处于三种状态中

+ followers  : 所有的节点在启动时都是 follower状态，如果在一定时间之后follower 没有收到leader的信息，就会自动切换为 candidate 角色然后发起选举并将选举信息推送给其他follower，如果收到 majority 的响应，那么就会切换为leader。
+ candidate ： follower 和 leader的中间角色
+ leader  ： 负责处理client的请求，并处理 replicated log的处理，leader 会有任期(term)，在一个`term`完成

![img](https://img2018.cnblogs.com/blog/1089769/201812/1089769-20181216202049306-1194425087.png) 

  term（任期）以选举（election）开始，然后就是一段或长或短的稳定工作期（normal Operation）。 任期是递增的，这就充当了逻辑时钟的作用；另外，term  还会有一种极端的场景出现，就是脑裂，就是说没有选举出leader就结束了，然后会发起新的选举，后面会解释这种*split vote*的情况。





## Safety 要求

  在上面提到只要日志被复制到majority节点，就能保证不会被回滚，即使在各种异常情况下，这根leader election提到的选举约束有关。在这一部分，主要讨论raft协议在各种各样的异常情况下如何工作的。

  衡量一个分布式算法，有许多属性，如

- safety：nothing bad happens,
- liveness： something good eventually happens.

  在任何系统模型下，都需要满足safety属性，即在任何情况下，系统都不能出现不可逆的错误，也不能向客户端返回错误的内容。比如，raft保证被复制到大多数节点的日志不会被回滚，那么就是safety属性。而raft最终会让所有节点状态一致，这属于liveness属性。

  raft协议会保证以下属性
![img](https://img2018.cnblogs.com/blog/1089769/201812/1089769-20181216202333639-30919755.png)

### Election safety

  选举安全性，即任一任期内最多一个leader被选出。这一点非常重要，在一个复制集中任何时刻只能有一个leader。系统中同时有多余一个leader，被称之为脑裂（brain split），这是非常严重的问题，会导致数据的覆盖丢失。在raft中，两点保证了这个属性：

- 一个节点某一任期内最多只能投一票；
- 只有获得majority投票的节点才会成为leader。

  因此，**某一任期内一定只有一个leader**。

### log matching

  很有意思，log匹配特性， 就是说如果两个节点上的某个log entry的log index相同且term相同，那么在该index之前的所有log entry应该都是相同的。如何做到的？依赖于以下两点

- If two entries in different logs have the same index and term, then they store the same command.
- If two entries in different logs have the same index and term, then the logs are identical in all preceding entries.

  首先，leader在某一term的任一位置只会创建一个log entry，且log entry是append-only。其次，consistency check。leader在AppendEntries中包含最新log entry之前的一个log 的term和index，如果follower在对应的term index找不到日志，那么就会告知leader不一致。

  在没有异常的情况下，log matching是很容易满足的，但如果出现了node crash，情况就会变得负责。比如下图![img](https://img2018.cnblogs.com/blog/1089769/201812/1089769-20181216202408734-1760694063.png)

  **注意**：上图的a-f不是6个follower，而是某个follower可能存在的六个状态

  leader、follower都可能crash，那么follower维护的日志与leader相比可能出现以下情况

- 比leader日志少，如上图中的ab
- 比leader日志多，如上图中的cd
- 某些位置比leader多，某些日志比leader少，如ef（多少是针对某一任期而言）

  当出现了leader与follower不一致的情况，leader强制follower复制自己的log

> To bring a follower’s log into consistency with its own, the leader must find the latest log entry where the two logs agree, delete any entries in the follower’s log after that point, and send the follower all of the leader’s entries after that point.

  leader会维护一个nextIndex[]数组，记录了leader可以发送每一个follower的log index，初始化为eader最后一个log index加1， 前面也提到，leader选举成功之后会立即给所有follower发送AppendEntries RPC（不包含任何log entry， 也充当心跳消息）,那么流程总结为：

> s1 leader 初始化nextIndex[x]为 leader最后一个log index + 1
> s2 AppendEntries里prevLogTerm prevLogIndex来自 logs[nextIndex[x] - 1]
> s3 如果follower判断prevLogIndex位置的log term不等于prevLogTerm，那么返回 false，否则返回True
> s4 leader收到follower的恢复，如果返回值是True，则nextIndex[x] -= 1, 跳转到s2. 否则
> s5 同步nextIndex[x]后的所有log entries

### leader completeness vs elcetion restriction

  leader完整性：如果一个log entry在某个任期被提交（committed），那么这条日志一定会出现在所有更高term的leader的日志里面。这个跟leader election、log replication都有关。

- 一个日志被复制到majority节点才算committed
- 一个节点得到majority的投票才能成为leader，而节点A给节点B投票的其中一个前提是，B的日志不能比A的日志旧。下面的引文指处了如何判断日志的新旧

> voter denies its vote if its own log is more up-to-date than that of the candidate.

> If the logs have last entries with different terms, then the log with the later term is more up-to-date. If the logs end with the same term, then whichever log is longer is more up-to-date.

  上面两点都提到了majority：commit majority and vote majority，根据Quorum，这两个majority一定是有重合的，因此被选举出的leader一定包含了最新的committed的日志。

  raft与其他协议（Viewstamped Replication、mongodb）不同，raft始终保证leade包含最新的已提交的日志，因此leader不会从follower catchup日志，这也大大简化了系统的复杂度。