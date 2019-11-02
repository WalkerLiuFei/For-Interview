# Redis 面试题

1. **Redis 哨兵集群模式有什么缺点**

   网络分区问题，如果一半以上的哨兵和两个从库连得上，另外一波一个分区，它们之间相互割裂，那么贪

2. **如何解决 主从Redis 延时备份问题，数据一致性稳定确定**

  1. 保证slave将数据写入以后，master再将数据响应
  2. 

3. Redis Scan 实现的原理,增量式迭代

  1. 增量式的迭代，一次性可以返回一部分要查询的内容

4. Redis的分布式并发竞争问题如何解决

   1. 由于Redis是单线程的，所以其本身并不负责解决并发竞争问题，需要业务代码自行控制
   2. 通过一个独占的排他锁，在修改键值时需要首先获得锁、
   3. 使用Redis的 CAS 操作 ： https://redis.io/topics/transactions#optimistic-locking-using-check-and-set
     1. 通过在事务中增加`watch key` 命令，当 key值发生变化后，事务中断。

5. **Redis缓存穿透 ： 当查询一个数据时，缓存中数据不存在，然后下沉到持久化的DB中查询，如果DB中也不存在，一般来说缓存不了这个键值。如果一直查询这个键，自然增大了持久化DB的压力。**

  1. 有很多种方法可以有效地解决缓存穿透问题，最常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。另外也有一个更为简单粗暴的方法（我们采用的就是这种），如果一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。

6. **Redis缓存雪崩 ： 大量的键值设置了同样的过期时间，然后在某一时刻同时过期，或者在缓存中读入大量的非热点数据将热点数据挤出缓存**

  1. 考虑用加锁或者队列的方式保证缓存的单线 程（进程）写，从而避免失效时大量的并发请求落到底层存储系统上。这里分享一个简单方案就时讲缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

7. **Redis 缓存击穿 ： 当一个热点的键值过期，或者从缓存中被挤出，因为这是个热点键值，瞬时的大量并发可能将后端DB压垮**

  1. 解决办法 ： 当读取不到热点键值对时，首先通过`SETNX`命令拿到一个分布式锁，获取锁成功后再去后端 DB Load 数据并缓存起来，如果获取锁失败的话就等待 （因为load起数据会很快）

    ```
    
    public String get(key) {
          String value = redis.get(key);
          if (value == null) { //代表缓存值过期
              //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
    		  if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
                   value = db.get(key);
                          redis.set(key, value, expire_secs);
                          redis.del(key_mutex);
                  } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                          sleep(50);
                          get(key);  //重试
                  }
              } else {
                  return value;      
              }
    }
    ```

  2. 永远不过期！

  3. 使用hystrix 做资源池隔离保护主线程,即只允许部分请求进来，其他请求直接响应失败，这样可以保证服务可用不消耗系统资源。

8. 了解Redis事务的CAS操作吗

  1. https://redis.io/topics/transactions#optimistic-locking-using-check-and-set
  2. CAS操作： 通过 一个 事务 将`WATCH KEY`   和`UPDATE Key` 包装起来

9. 缓存机器增删如何对系统影响最小，一致性哈希的实现

  1. 

10. Redis持久化的几种方式，优缺点是什么，怎么实现的

11. RDB持久化优点

   1. 适合做缓存快照
   2. 因为是一个单一文件，所以很好恢复，并且易于传输
   3. 父进程只需要fork 子进程，子进程会将剩余工作完成
   4. 相较于AOF，RDB更容易重启时恢复

12. RDB持久化缺陷

   1. RDB相较于AOF的增量式备份，更容易丢失数据
   2. 如果CPU Performance 不好，或者数据量很大，会导致一次子进程fork 花费大量时间，从而导致server 对外是不可用的。

13. AOF优势

   1. 增量式持久化，将数据丢失降到最小
   2. 通过重写可以将command log 文件降到最小
   3. 文件中命令格式化的很好，易读性好

14. AOF劣势

   1. AOF 文件相对较大
   2. 根据确切的fsync策略，AOF可能比RDB慢。 通常，在将fsync设置为每秒的情况下，性能仍然很高，并且在禁用fsync的情况下，即使在高负载下，它也应与RDB一样快。 即使在巨大的写负载的情况下，RDB仍然能够提供有关最大延迟的更多保证。

15. Redis的缓存过期策略

   1. 定期过期和惰性过期
   2. 惰性过期是使用最多的，即只有在read的时候才会判断键值是否过期

16. Redis内存淘汰策略

   1. noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。
   2. allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
   3. allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
   4. volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
   5. volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
   6. volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

17. redis集群，高可用，原理

   1. Redis 集群通过哨兵监控 一个 分片 shard 下面一个`master`节点和多个`slave`节点当master节点宕机后，哨兵sential选择一个 slave 作为master

18. MYSQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

   14. 限定Redis内存max值即可，Redis有自己的一套淘汰策略

19. 用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次

   1. https://github.com/aCoder2013/blog/issues/26

   2. 注意在 键值过期后再去 INCR

   3. 也可以利用 

     ```java
     try (Jedis redis = getRedis()) {
     	Long count = redis.incrBy(key.getBytes(), val);
     	if (count == val) {
     	    redis.expire(key, exp);
     	}
     }
     ```

     方式进行设置过期时间

   ```
   result = SETNX login_{user-id} 0 3600
   if result == 0:
   	return success
   	
   ```

   1. 

20. Redis的数据淘汰策略

21. Redis 与epoll 多路复用IO

22. Redis的数据结构分析

23. Redis主从复制原理及无磁盘复制分析

24. Redis管道模式详解

25. Redis缓存与数据库一致性问题解决方案

26. 基于Redis实现分布式锁实战

27. 图解Redis中的AOF和RDB持久化策略的原理

28. Redis读写分离架构实践

29. Redis哨兵架构及数据丢失问题分析

30. Redis  luster数据分布算法之 Hash Slot

31. Redis使用常见问题及性能优化思路

32. Redis高可用及高伸缩架构实战

33. 缓存击穿，缓存雪崩预防策略

34. Redis批量查询优化

35. Redis高性能集群之Twemproxy  or  codis





