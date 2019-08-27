# Java 面试题(一)

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
   3. `newFixedThreadPool` : 执行长期的任务，性能好, 不会直接释放资源。如果数量大，对服务器性能有要求。
   4. `NewScheduledThreadPool` : 周期性执行任务的场景
   
4. **分析线程池的实现原理和线程的调度过程** 
1. 当线程池小于corePoolSize时，**新提交任务**将创建一个新线程执行任务，即使此时线程池中的`workQueue` 中存在排队的任务。
   2. 当线程池达到corePoolSize时，新提交任务将被放入`workQueue`中，等待线程池中任务调度执行.
   3. 当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务。
   4. 当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutioException.
   5. 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
   6. 当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭
   
5. **线程池如何调优** 
1. 参考
      1. [Java线程池分析及策略优化](https://www.jianshu.com/p/896b8e18501b)
      2. [记一次线程池调优经历](https://www.cnblogs.com/superfj/p/8313469.html)
      3. [Java线程池分析及策略优化](https://www.jianshu.com/p/896b8e18501b) 
   2. 最大线程数要 >=  CPU 核心数
   3. 对于最大的线程数的设置 ，一旦服务器成为瓶颈，向服务器对于CPU密集型或IO密集型的机器增加线程数实际会降低整体的吞吐量；
   4. 应尽量避免频繁的出现线程的创建和回收，因为这非常消耗CPU资源，所以对于最小线程池的数量，应该可以满足日常任务。
   5. IO密集型可以考虑多些线程来平衡CPU的使用，CPU密集型可以考虑少些线程减少线程调度的消耗，所以对于线程池中的 keepAliveTime的调整也要适度。
   6. Java线程池调度策略一直等到任务队列满才开始创建新线程，这个不是我希望看到的。我们希望的是：可以给等待队列设置一个阈值，一旦触及这个阈值马上创建新的线程，防止任务出现过度堆积。
   
6. **线程池的最大线程数目根据什么确定**

   1. 参考 
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
   2. JVM反射 : 运行期通过读取 方法区来进行代理，效率低。

8. **HashMap的并发问题**

   1. HashMap并不是线程安全的，可以使用ConcurrentHashMap来进行替换

9. **了解LinkedHashMap的应用吗** 

   1. LinkedHashMap是有序的，插入有序的

10. **反射的原理，反射创建类实例的三种方式是什么?** 

    1. 发射原理
       1. [反射原理](http://www.fanyilun.me/2015/10/29/Java%E5%8F%8D%E5%B0%84%E5%8E%9F%E7%90%86/)
       2. [关于反射调用方法的一个log](https://rednaxelafx.iteye.com/blog/548536)
       3. 

11. **cloneable接口实现原理，浅拷贝or深拷贝** 

    1. 一般不用Cloneable，一般使用 序列化和反序列化的方式来对对象进行拷贝
    2. Spring 的`BeanUtils` 工具是使用的反射拿到所有的field 值来进行的。

12. **Java NIO，AIO使用,原理**  

    1. 参考 
       1. [java nio及操作系统底层原](https://blog.csdn.net/u014507083/article/details/73784898)
       2. [Java  NIO,AIO,BIO的区别](https://blog.csdn.net/skiof007/article/details/52873421)  
    2. NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统
    3. 
    4. **BIO 和 NIO,AIO 区别** 
       1. BIO 是流导向的，NIO 是缓冲导向的
          1. 在不对数据进行缓存的情况下，你是没办法对数据进行index操作的
          2. 缓冲导向是将数据已经buffer起来的。
       2. BIO是阻塞同步IO，NIO是非阻塞同步IO，AIO 是非堵塞异步IO
          1. NIO 相较于AIO而言，NIO是非堵塞的但是 R/W的处理还是需要Process 自己处理，而AIO是交给OS来进行 R/W操作
       3. 各自的应用场景 ：
          1. BIO ： 传统的小架构应用，处理少量的应用等
          2. NIO ： 处理连接数量多，且短的应用，如一些websocket应用等等,Reactor 模式
          3. AIO ： 处理大量连接，且IO 压力大的服务器，充分调用OS参与并发，如相册服务器。
    5. **NIO 可以通过 selector 使得可以利用一个thread来管理多个channels (网络链接或者files R/W) 但是同样的，需要字节维护好数据的IO操作的完整性。明显的，通过这样的方式，可以节省不少CPU资源，因为如果使用传统BIO的话，每个网络链接或者files R/W 分配一个thread然后处理一个IO操作，这样相对于NIO就能降低不少CPU的压力** 
       1. 场景1 ： 大量的链接，每个链接只读取少量数据
       2. 场景2 ： 长时间的P2P链接，比如websocket

13. TreeMap的实现原理

14. Arrays.sort(object[]) 使用 的 **TimSort** 的实现原理，**TimSort**有没有什么问题？

    1. 基本的实现原理 ：
       1. 扫描数组，确定其中的单调上升段和严格单调下降段，将严格下降段反转。我们将这样的段称之为run。
       2. 定义最小run长度，短于此的run通过插入排序合并为长度高于最小run长度；
       3. 反复归并一些相邻run，过程中需要**避免归并长度相差很大的run**，直至整个排序完成；
       4. 如何避免归并长度相差很大run呢， 依次将run压入栈中，若栈顶run X，run Y，run Z 的长度违反了**X>Y+Z 或 Y>Z** 则Y run与较小长度的run合并，并再次放入栈中。 依据这个法则，能够尽量使得大小相同的run合并，以提高性能。注意Timsort是稳定排序故只有相邻的run才能归并。
    2. TimSort 的Bug
       1. [TimSort的Bug](http://www.envisage-project.eu/proving-android-java-and-python-sorting-algorithm-is-broken-and-how-to-fix-it)
       2. [形式化方式找到的TimSort Bug](https://bindog.github.io/blog/2015/03/30/use-formal-method-to-find-the-bug-in-timsort-and-lunar-rover)

15. 介绍一下 1.7 以后开始支持的 **Fork / Join** 框架

    1. Fork / Join 的原理有两个 , **递归拆分** ，**工作窃取** 

    2. 流程

       1. 分割任务
       2. 第二步执行任务并合并结果

    3. 工作窃取算法 ： 工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。

    4. Fork/Join使用两个类来完成以上两件事情：

       - ForkJoinTask：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
         - RecursiveAction：用于没有返回结果的任务。
         - RecursiveTask ：用于有返回结果的任务。
       - ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。
