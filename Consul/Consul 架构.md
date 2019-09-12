# Consul 架构

## Consul架构中的名词

### Agent

代理是`Consul`集群的每个成员上的长时间运行守护程序。它是由正在运行的`Agent`启动的。`Agent`可以 以Client或者`Server` 的模式进行运行。所有的Agent都可以进行`DNS`或者`HTTP` ，并负责运行检查并保持服务同步。`Agent` 是整个`Consul`集群的核心.

### Client

客户端是将所有RPC转发到服务器的代理(agent)。客户是相对无状态的。客户端执行的唯一后台活动是参与 `LAN gossip pool` 。这具有最小的资源开销并且仅消耗少量的网络带宽。

### Server

运行`Server`模式的`Agent`，参与`Raft quorum`，维护集群的状态，响应RPC查询，与其他数据中心交互`WAN gossip`，转发查询到Leader或远程数据中心。

### data center  

数据中心的定义似乎是显而易见的，有一些细节是必须考虑的。例如，在EC2，多个可用性区域是否被认为组成了单一的数据中心？我们定义数据中心是在同一个网络环境中——私有的，低延迟，高带宽。这不包括基于公共互联网环境，但是对于我们而言，在同一个EC2的多个可用性区域会被认为是一个的数据中心。

### Consensus 

consensus，意味着leader election协议，以及事务的顺序。由于这些事务是基于一个有限状态机，consensus的定义意味着复制状态机的一致性。 

### Gossip

 consul是建立在`Serf`之上，提供了完成的`Gossip`协议，用于成员维护故障检测、事件广播。详细细节参见*gossip documentation*。这足以知道`gossip`是基于UDP协议实现随机的节点到节点的通信，主要是在UDP。

### LAN Gossip

指的是LAN gossip pool，包含位于同一个局域网或者数据中心的节点。

### WAN Gossip

指的是WAN gossip pool，只包含server节点，这些server主要分布在不同的数据中心或者通信是基于互联网或广域网的。

### RPC

远程过程调用。是允许client请求服务器的请求/响应机制。

### 架构图

<img src="https://img-blog.csdn.net/20160827175251975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="img" style="zoom: 25%;" />

从图中可以看到,`consul`支持多数据中心.多数据中心是`consul`的一大特性. 对于每个`datacenter`为在可用性和一致性之间求的一个平衡,一般一个datacenter由 3-5个server组成,但是client没有限制.所有的`agent` 都参与`gossip protocol` 

