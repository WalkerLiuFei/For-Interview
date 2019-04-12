## **java高频面试题如下（文末准备面试资料及答案免费领取）：**

**java基础**

1. **Arrays.sort实现原理和Collections.sort实现原理** 

   1. Collections.sort 最终是实现的是调用集合的自己的实现
   2. LinkList 是将链表转换为数组后调用Arrays.sort, ArrayList 就是直接调用的Arrays.sort
   3. Arrays.sort 使用的排序算法
      1. 如果是 原始类型数组的话，直接用的是**双轴快拍算法**
      2. 另外还有从 1.8开始的 **ForkJoinPool** 进行的排序算法
      3. 如果是Object[] 类型的则默认使用 **TimSort**
      4. 在JDK1.7以前我们几乎就是按照MergeSort来实现Arrays.sort()的。只是细节上略有差别。当集合小于7的时候我们使用插入排序。当集合大于7的时候，先通过递归分解，把集合分解成若干等于7的子集合。我们再采用插入排序来进行子集合的排序。

2. **foreach和while的区别(编译之后)** 

   1. foreach 只是 `for (int index = 0； index < len ； index++)` 的语法糖

3. **线程池的种类，区别和使用场景** 

   1. [参考](https://www.cnblogs.com/sachen/p/7401959.html)
   2. `newCacheThreadPool` : 执行很多短期异步的小程序或者负载较轻的服务器
   3. `newFixedThreadPool` : 执行长期的任务，性能好
   4. `NewScheduledThreadPool` : 周期性执行任务的场景

4. **分析线程池的实现原理和线程的调度过程** 

   1. 当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。
   2. 当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行
   3. 当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务
   4. 当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理
   5. 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
   6. 当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭

5. **线程池如何调优** 

   1. 参考
      1. [Java线程池分析及策略优化](https://www.jianshu.com/p/896b8e18501b)
      2. [记一次线程池调优经历](https://www.cnblogs.com/superfj/p/8313469.html)
      3. [Java线程池分析及策略优化](https://www.jianshu.com/p/896b8e18501b) 
   2. 最大线程数要 >=  CPU 核心数
   3. 对于最大的线程数的设置 ，一旦服务器成为瓶颈，向服务器对于CPU密集型或IO密集型的机器增加线程数实际会降低整体的吞吐量；
   4.  应尽量避免频繁的出现线程的创建和回收，因为这非常消耗CPU资源，所以对于最小线程池的数量，应该可以满足日常任务。
   5. IO密集型可以考虑多些线程来平衡CPU的使用，CPU密集型可以考虑少些线程减少线程调度的消耗，所以对于线程池中的 keepAliveTime的调整也要适度。
   6. Java线程池调度策略一直等到任务队列满才开始创建新线程，这个不是我希望看到的。我们希望的是：可以给等待队列设置一个阈值，一旦触及这个阈值马上创建新的线程，防止任务出现过度堆积。

6. **线程池的最大线程数目根据什么确定**

   1.  参考 
      1. [如何合理地估算线程池大小](http://ifeve.com/how-to-calculate-threadpool-size) 
      2. [Java计算线程CPU 使用率](https://my.oschina.net/xpbob/blog/624719) 
   2. 如果是CPU密集型的应用，设置为 N+1 
   3. 如果是IO密集型的应用，设置为 2N + 1
   4. **最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）* CPU数目** 
   5. 使用线程池未必就比使用线程池高效，比如Redis就是单线程的，因为Redis的操作基本都是IO的内存操作，使用单线程减少了**锁管理**和**线程上线文切换**  ，所以单线程反而是对CPU的更加充分的利用。同样的，我们把Redis的行为方式套用到 4 中的公式中，由于非CPU使用时间（线程等待时间） 远大于线程CPU使用时间，所以其比值接近于0 。从Redis的CPU使用方式也恰好证明了我们最佳线程数目的正确性。

7. **动态代理的几种方式** 

   1. ASM 字节增强： 在编译期直接生成
      1. [Java字节码处理框架ASM设计思想解析](https://www.jianshu.com/p/26e99d39b3fb) 
      2. CGLIB原理：动态生成一个要代理类的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用java反射的JDK动态代理要快。
      3. CGLIB底层：使用字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。
      4. CGLIB 无法代理 final方法，因为CGLIB 是通过生成子类来生成AOP的。
      5. 
   2. JVM反射 : 运行期通过读取 方法区来进行代理，效率低。

8. **HashMap的并发问题**

   1. HashMap并不是线程安全的，可以使用ConcurrentHashMap来进行替换

9. **了解LinkedHashMap的应用吗** 

   1. LinkedHashMap是有序的，插入有序的

10. **反射的原理，反射创建类实例的三种方式是什么?** 

    1. 发射原理
       1. [关于反射调用方法的一个log](https://rednaxelafx.iteye.com/blog/548536)
       2. 

11. cloneable接口实现原理，浅拷贝or深拷贝

12. Java NIO使用

13. hashtable和hashmap的区别及实现原理，hashmap会问到数组索引，hash碰撞怎么解决

14. arraylist和linkedlist区别及实现原理

15. 反射中，Class.forName和ClassLoader区别

16. String，Stringbuffer，StringBuilder的区别？

17. 有没有可能2个不相等的对象有相同的hashcode

18. 简述NIO的最佳实践，比如netty，mina

19. TreeMap的实现原理

20. Arrays.sort(object[]) 使用 的 **TimSort** 的实现原理，**TimSort**有没有什么问题？

    1. 基本的实现原理 ：
       1. 扫描数组，确定其中的单调上升段和严格单调下降段，将严格下降段反转。我们将这样的段称之为run。
       2. 定义最小run长度，短于此的run通过插入排序合并为长度高于最小run长度；
       3. 反复归并一些相邻run，过程中需要**避免归并长度相差很大的run**，直至整个排序完成；
       4. 如何避免归并长度相差很大run呢， 依次将run压入栈中，若栈顶run X，run Y，run Z 的长度违反了**X>Y+Z 或 Y>Z** 则Y run与较小长度的run合并，并再次放入栈中。 依据这个法则，能够尽量使得大小相同的run合并，以提高性能。注意Timsort是稳定排序故只有相邻的run才能归并。
    2. TimSort 的Bug
       1. [TimSort的Bug](http://www.envisage-project.eu/proving-android-java-and-python-sorting-algorithm-is-broken-and-how-to-fix-it)
       2. [形式化方式找到的TimSort Bug](https://bindog.github.io/blog/2015/03/30/use-formal-method-to-find-the-bug-in-timsort-and-lunar-rover)

21. 介绍一下 1.7 以后开始支持的 **Fork / Join** 框架


**JVM相关**

1. 类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，他们的执行顺序
2. JVM内存分代
3. Java 8的内存分代改进
4. JVM垃圾回收机制，何时触发MinorGC等操作
5. jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等
6. 你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1
7. 新生代和老生代的内存回收策略
8. Eden和Survivor的比例分配等
9. 深入分析了Classloader，双亲委派机制
10. JVM的编译优化
11. 对Java内存模型的理解，以及其在并发中的应用
12. 指令重排序，内存栅栏等
13. OOM错误，stackoverflow错误，permgen space错误
14. JVM常用参数
15. tomcat结构，类加载器流程
16. volatile的语义，它修饰的变量一定线程安全吗
17. g1和cms区别,吞吐量优先和响应优先的垃圾收集器选择
18. 说一说你对环境变量classpath的理解？如果一个类不在classpath下，为什么会抛出ClassNotFoundException异常，如果在不改变这个类路径的前期下，怎样才能正确加载这个类？
19. 说一下强引用、软引用、弱引用、虚引用以及他们之间和gc的关系

**JUC/并发相关**

1. ThreadLocal用过么，原理是什么，用的时候要注意什么
2. Synchronized和Lock的区别
3. synchronized 的原理，什么是自旋锁，偏向锁，轻量级锁，什么叫可重入锁，什么叫公平锁和非公平锁
4. concurrenthashmap具体实现及其原理，jdk8下的改版
5. 用过哪些原子类，他们的参数以及原理是什么
6. cas是什么，他会产生什么问题（ABA问题的解决，如加入修改次数、版本号）
7. 如果让你实现一个并发安全的链表，你会怎么做
8. 简述ConcurrentLinkedQueue和LinkedBlockingQueue的用处和不同之处
9. 简述AQS的实现原理
10. countdowlatch和cyclicbarrier的用法，以及相互之间的差别?
11. concurrent包中使用过哪些类？分别说说使用在什么场景？为什么要使用？
12. LockSupport工具
13. Condition接口及其实现原理
14. Fork/Join框架的理解
15. jdk8的parallelStream的理解
16. 分段锁的原理,锁力度减小的思考

**Spring**

1. Spring AOP与IOC的实现原理
2. Spring的beanFactory和factoryBean的区别
3. 为什么CGlib方式可以对接口实现代理？
4. RMI与代理模式
5. Spring的事务隔离级别，实现原理
6. 对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的？
7. Mybatis的底层实现原理
8. MVC框架原理，他们都是怎么做url路由的
9. spring boot特性，优势，适用场景等
10. quartz和timer对比
11. spring的controller是单例还是多例，怎么保证并发的安全

**分布式相关**

1. Dubbo的底层实现原理和机制
2. 描述一个服务从发布到被消费的详细过程
3. 分布式系统怎么做服务治理
4. 接口的幂等性的概念
5. 消息中间件如何解决消息丢失问题
6. Dubbo的服务请求失败怎么处理
7. 重连机制会不会造成错误
8. 对分布式事务的理解
9. 如何实现负载均衡，有哪些算法可以实现？
10. Zookeeper的用途，选举的原理是什么？
11. 数据的垂直拆分水平拆分。
12. zookeeper原理和适用场景
13. zookeeper watch机制
14. redis/zk节点宕机如何处理
15. 分布式集群下如何做到唯一序列号
16. 如何做一个分布式锁
17. 用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗
18. MQ系统的数据如何保证不丢失
19. 列举出你能想到的数据库分库分表策略；分库分表后，如何解决全表查询的问题。

**算法和数据结构以及设计模式**

1. 海量url去重类问题（布隆过滤器）
2. 数组和链表数据结构描述，各自的时间复杂度
3. 二叉树遍历
4. 快速排序
5. BTree相关的操作
6. 在工作中遇到过哪些设计模式，是如何应用的
7. hash算法的有哪几种，优缺点，使用场景
8. 什么是一致性hash
9. paxos算法
10. 在装饰器模式和代理模式之间，你如何抉择，请结合自身实际情况聊聊
11. 代码重构的步骤和原因，如果理解重构到模式？

**数据库**

1. MySQL InnoDB存储的文件结构
2. 索引树是如何维护的？
3. 数据库自增主键可能的问题
4. MySQL的几种优化
5. mysql索引为什么使用B+树
6. 数据库锁表的相关处理
7. 索引失效场景
8. 高并发下如何做到安全的修改同一行数据，乐观锁和悲观锁是什么，INNODB的行级锁有哪2种，解释其含义
9. 数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁

**Redis&缓存相关**

1. Redis的并发竞争问题如何解决了解Redis事务的CAS操作吗
2. 缓存机器增删如何对系统影响最小，一致性哈希的实现
3. Redis持久化的几种方式，优缺点是什么，怎么实现的
4. Redis的缓存失效策略
5. 缓存穿透的解决办法
6. redis集群，高可用，原理
7. mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据
8. 用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次
9. redis的数据淘汰策略

**网络相关**

1. http1.0和http1.1有什么区别
2. TCP/IP协议
3. TCP三次握手和四次挥手的流程，为什么断开连接要4次,如果握手只有两次，会出现什么
4. TIME_WAIT和CLOSE_WAIT的区别
5. 说说你知道的几种HTTP响应码
6. 当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤
7. TCP/IP如何保证可靠性，数据包有哪些数据组成
8. 长连接与短连接
9. Http请求get和post的区别以及数据包格式
10. 简述tcp建立连接3次握手，和断开连接4次握手的过程；关闭连接时，出现TIMEWAIT过多是由什么原因引起，是出现在主动断开方还是被动断开方。

**其他**

1. maven解决依赖冲突,快照版和发行版的区别
2. Linux下IO模型有几种，各自的含义是什么
3. 实际场景问题，海量登录日志如何排序和处理SQL操作，主要是索引和聚合函数的应用
4. 实际场景问题解决，典型的TOP K问题
5. 线上bug处理流程
6. 如何从线上日志发现问题
7. linux利用哪些命令，查找哪里出了问题（例如io密集任务，cpu过度）
8. 场景问题，有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。
9. 用三个线程按顺序循环打印abc三个字母，比如abcabcabc。
10. 常见的缓存策略有哪些，你们项目中用到了什么缓存系统，如何设计的
11. 设计一个秒杀系统，30分钟没付款就自动关闭交易（并发会很高）
12. 请列出你所了解的性能测试工具
13. 后台系统怎么防止请求重复提交？