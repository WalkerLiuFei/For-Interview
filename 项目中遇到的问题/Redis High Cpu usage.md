# CPU high CPU usage

## 问题描述

Redis的使用非常高，能达到100%，Redis就是一个单机Redis ，持久化策略是`AOF`,100%增长率时进行重写,连接大概也只有50个。

Redis存储了大量的 Set集合。而筛选集合

## 问题解决过程

1. 因为现在用 AOF sync `every second` 的策略，怀疑是这里的问题。
2. 但是我动态的通过`CONFIG SET APPENDONLY NO` 命令关闭AOF持久化后，CPU并没有下降。
3. 现在将`RDB`和`AOF`持久化全部关闭以后，cpu 使用率依旧没有下降。

最后发现还是业务的原因，目前每秒大概处理2000条消息，每条消息完全处理大概需要25次的Redis 读写操作。这样子就导致redis-server 单机每秒要完成 50000 QPS，我测试了一下，因为包含了很多`SMEMBERS` 和`LRANGE` 这样的操作，5w QPS差不多是单机CPU的极限了。





