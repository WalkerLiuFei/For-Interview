1. private修饰的方法可以通过反射访问，那么private的意义是什么

2. Java类初始化顺序

3. 方法区和永久区的理解以及它们之间的关系

   1. 方法区是JVM规范里要求的，永久区是Hotspot虚拟机对方法区的具体实现。方法区是规范，永久区是实现

4. 一个java文件有3个类，编译后有几个class文件

   1. 有几个类就有几个 class文件，内部类也一样

5. 局部变量使用前需要显式地赋值，否则编译通过不了，为什么这么设计

   1. 成员变量可以不经过初始化，在类加载过程的准备阶段即可给它赋予默认值，但局部变量使用前需要显式赋予初始值。因为成员变量的赋值和取值访问顺序有不可确定性，所以编译器无法对其进行判断。但是对于局部变量而言其赋值和访问的顺序是确定的，为了防止程序员出现错误，编译器实际上是“帮助”报错的

6. ReadWriteLock读写之间互斥吗

   1. 要看加什么锁，是读还是写操作

7. Semaphore拿到执行权的线程之间是否互斥

   1. 不会互斥，Semaphore这个就是设计用来多线程访问的。

8. 写一个你认为最好的单例模式

   1. 双检锁单例模式

      ```java
      if (instance == null){
          synchronized(Singleton.class){
              if (instance == null){
                  instance = new Singleton();
              }
          }
          return instance
      }
      ```

9. B树和B+树是解决什么样的问题的，怎样演化过来，之间区别

   1. [B树](https://zh.wikipedia.org/wiki/B%E6%A0%91) 

   2. B树是一个自平衡树，查找时间是 O(lg N). B树适用于磁盘查找。

   3. B+ 相对于B树的优势

      1. B+ 树在内部节点上面只存Key,这样在将磁盘数据读取到内存中时，可以映射更多数据
      2. 由于B+树在内部节点中不存数据，所有B+树在读取数据时，可以线性的直接读取到叶子节点

      ![B and B+ tree](https://i.stack.imgur.com/l6UyF.png)

10. 写一个死锁

11. cpu 100%怎样定位
    1. top -c  显示进程运行信息列表，然后按P 进入CPU排序
    2. top -Hp <PID> 显示一个进程的所有运行线程，键入P 进入CPU排序
    3. printf "%x\n" PID  : 转为 16进制 ox<PID>
    4. jstack PID| grep ‘0xPID’ -C5 --color : 打印线程堆栈信息 

12. String a = "ab" String b = "a" + "b" a == b 是否相等，为什么    
    1. 是相等的，因为 String b = "a" + "b" 在编译器已经被编译器优化为 `String b = "ab"`  。所以 a == b 。
    2. 如果编译器 String b = "a" + "b" 编译为 String b = "a".concat("b") 那么他们不相等

13. 新的任务提交到线程池，线程池是怎样处理
    1. 线程池中的工作线程是否饱和？ 未饱和，创建一个县城执行提交的任务
    2. 工作线程已经饱和，提交到工作队列等待
    3. 如果等待的工作队列也满了，则交给饱和策略处理这个任务

14. AQS和CAS原理
    1. AQS(AbstractQueuedSychronizer) ,AQS 是 Java整个并发包`java.util.concurrent.locks` 的核心，ReentrantLock,CountDownLatch,Semaphore 等等都是以用它作为`线程管理`的。AQS将所有的线程放在FIFO队列中，开放 tryLock和tryRelease给继承使用。
    2. CAS(Compare and Set) , 假设有三个值，内存值 V，旧的预期值 A,要修改的值B，当 预期值A和内存值V相同时，才会将内存值修改为B并返回True，否则什么都不做并返回False.整个比较并替换是一个原子操作。**CAS操作一定要配合volatile修饰** ，这样才能保证每次拿到的变量是主内存中最新的响应值，否则旧的预期值A对某条线程来说，永远是一个不变的值A。
       1. 循环时间长开销大
       2. 只能保证一个共享变量的原子操作
       3. ABA问题 ： 如果CAS操作本身不满足原子性，就会出现ABA问题，
          1. 线程1 查询A的值为a，与旧值a比较，
          2. 线程2 查询A的值为a，与旧值a比较，相等，更新为b值
          3. 线程2 查询A的值为b，与旧值b比较，相等，更新为a值
          4. 线程1 相等，更新B的值为c

15. volatile作用，指令重排相关

    1. volatile 保证变量的访问的实时可见性，当一个变量存在多个线程同时访问的风险时一般都要加上这个volatile修饰符
    2. 防止指令重排，一般编码时，这个特性不是使用volatile的优先考虑的地方

16. AOP和IOC原理

    1. IOC 利用反射创建对象保存到Spring container 里面，使用时直接获取拿就可以
    2. AOP目前都是通过字节码增强的方式来实现的，相当于是对
