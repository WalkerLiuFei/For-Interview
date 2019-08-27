1. 类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，他们的执行顺序
   1. 父类静态字段
   2. 父类初始化代码块
   3. 静态字段
   4. 静态初始代码块
   5. 字段
   6. 初始化块
   7. 构造函数

2. **Java 8的内存分代改进** 
   1. Java 将永久区移除，使用**元空间**来存储类元数据信息，元空间相对于永久代来说，**元空间的数据存储在本地内存中**，相对于永久代的大小限制于JVM堆内存的大小，显而易见元空间的能够存放的数据更多。
   2. 去除永久代的原因：（1）为了HotSpot与JRockit的融合；（2）永久代大小不容易确定，PermSize指定太小容易造成永久代OOM，与老年代没关系。

3. **JVM垃圾回收机制，何时触发MinorGC等操作，何时触发Major GC, 何时触发Full GC** 
   1. [内存管理白皮书](https://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf) 
   2. Minor GC 
      1. Eden区满了，无法为新的object 分配空间
      2. Minor GC后，Eden区域是空的
   3. Major GC
      1. 大部分的Major GC都是由Minor  GC 引起的
      2. `System.GC()` 也会引起一个Minor GC

4. jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等

   1. 对象是如何晋升到老年代的

      1. 对象首先进入Eden区。
      2. 一次Minor GC后存活的到Survivor区，在Survivor区存活超过  参数`MaxTenuringThreshold` 的对象，晋升到老年代。该值默认是15，Survivor空间中相同年龄的对象的大小总和大于Survivor空间的一半，则年龄大于或等于该年龄的对象都会晋升到老年代。

   2. 几个重要的 JVM 参数

      |          参数          | default |              描述              |
      | :--------------------: | :-----: | :----------------------------: |
      |  -XX:+PrintGCDetails   |  false  | 开关参数，打印虚拟机GC详细日志 |
      |          -Xmx          | 不确定  |        虚拟机最大堆内存        |
      |          -Xms          | 不确定  |        虚拟机最小堆内存        |
      |      -XX:NewRatio      |    2    |      老年代和新生代的比例      |
      |   -XX:SurvivorRatio    |         |     设置Eden区和Survivor去     |
      |  -XX:ThreadStackSize   |         |          Thread Stack          |
      | -XX:+DisableExplicitGC |         |          关闭显式的GC          |
      |  -XX:+AggressiveOpts   |         |            加快编译            |
      |  -XX:+UseBiaseLocking  |         |          锁性能的改善          |

   3. `java -XX:+PrintFlagsFinal -version | grep HeapSize` ： 输出堆内存大小

5. 你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1

6. JVM  出现Full GC 很频繁，怎么排查这种问题 ？

   1. dump 出堆栈信息(`jmap -dump:format=b file=xxxx.bin pid`)，然后用可视化工具排查 

7. 深入分析了Classloader，双亲委派机制

   1. Bootstrap 类加载器，加载java、javax、sun等开头的类，是C++
   2. Extension Classloader 加载jdk扩展类库
   3. System Classloader ： 开发者可以直接使用这个classloader，可以直接使用`Classloader.getSystemClassloader` 来使用。这个类加载器用来加载`classpath` 指定路径的类库
   4. 双亲委派原则：
      1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器	去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归。
      2. 优势 ：**避免类的重复加载** 

8. JVM的编译优化

9. 能不能自己写个类叫`java.lang.System`？

   1. 通常不可以，但可以采取另类方法达到这个需求。 为了不让我们写System类，类加载采用委托机制，这样可以保证爸爸们优先，爸爸们能找到的类，儿子就没有机会加载。而System类是Bootstrap加载器加载的，就算自己重写，也总是使用Java系统提供的System，**自己写的System类根本没有机会得到加载。**

10. 指令重排序，内存栅栏等

   1. 指令重排序 ： 在java --> .class文件，.class --> 汇编，.class---> CPU指令期间，都有可能发生指令重排序，java指令重排序的最低保证是，as if serial，即在单个线程内，看起来代码是顺序运行的，但是在其他线程中，代码运行的顺序就不好说了。
   2. 内存屏障
      1. [参考](http://ifeve.com/memory-barriers-or-fences/)
      2. 一旦内存数据被推送到CPU缓存，就会有消息协议来确保所有的缓存会对所有的共享数据同步并保持一致。这个使内存数据对CPU核可见的技术被称为**内存屏障或内存栅栏**。
      3. 内存屏障提供的两个用呢个
         1. 确保从另一个CPU来看的屏障的两边的所有指令都是正确的程序顺序，而保持程序的外部可见性
         2. 可以实现内存数据可见性，确保数据会同步到CPU缓存子系统
      4. 内存屏障举例
         1. Store Barrier ： x86的`sfence`指令，强制所有在`sfence`指令之前的`store`指令都在该store指令前执行，并把store缓冲区的数据都缓冲到CPU缓存。
         2. Load Barrier :x86 中的`ifence` ，强制所有在`ifence`之后的`load`指令，都在该`load`屏障指令执行之后被执行，并且一直等到`load`缓冲区被该CPU读完才能执行之后的`load`指令。这使得从其它CPU暴露出来的程序状态对该CPU可见，这之后CPU可以进行后续处理。
         3. `full Barrier`  x86 上的 `mfence` 指令，结合了以上两条指令的功能
      5. **volatile**变量在写操作之后会插入一个store屏障，在读操作之前会插入一个load屏障。一个类的**final**字段会在初始化后插入一个store屏障，来确保final字段在构造函数初始化完成并可被使用时可见
      6. 应用中的 `lock` 会结合一个 `mfence` 指令 