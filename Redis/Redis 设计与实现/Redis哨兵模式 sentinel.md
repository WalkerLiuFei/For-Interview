# Redis Sentinel

Redis 哨兵模式是Redis的高可用解决方案,由一个或者多个`sentinel` （哨兵）实例组成的`Sentinel`系统，`Sentinel`监控任意多个主服务器，以及主服务器下面的所有从服务器，当被监视的主服务器下线时，自动将下线的主服务器下面的从服务器升级为主服务器。



### 启动初始化Sential

`启动一个redis-sentinel /path/config/sentinel.conf` 

Sentinel 只是一个运行在特殊环境下的Redis服务器，所以启动Sentinel的第一步，就是初始化一个普通的`Redis`服务器。

默认运行在端口`26379`。

对于普通模式下的Redis命令 `sentinel`是无法运行的

`sentinel` 中的`masters`和`slaves` 两个字典值存储了管理的信息。

### Sential管理主从服务器

Sential 通过会以10秒的间隔向主从服务器发送 `INFO`命令来获取其配置，通过`PING /PONG`命令来模拟心跳。

### 主观下线状态客观下线状态

主观下线状态是指：当多个`sentinel`在监控同一个主从集群时，其中`sentinel-1` 通过自己配置的超时时间认为认为master已经 下线了，那么这时候`sentinel-1`将master的下线状态设置为 主观下线。

当`sentinel`认为master下线后，就开始询问其他`sentinel`master的状态，当收到其他的`sentinel`认为`master`下线达到阈值以后，`sentinel`会将master设置为可观下线状态，并进行故障状态转移。

### 领头的Sentinel进行故障转移

当sentinel集群认为master下线后，就需要推选出一个 leader进行master的故障转移并同步未执行的命令，这个时候为保持数据一致性就需要一个`sentinel`-leader  来进行故障转移。

这个leader的选举过程其实就是`RAFT`共识协议的实践。

在 leader-sentinel节点选出一个状态良好，数据完整的(可以同过响应市场判断)，选出一个slave服务器之后像这个从服务器发送`SLAVEOF no one` 命令，将其转换为主服务器



