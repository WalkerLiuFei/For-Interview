# Synchronized 实现以及原理

**三种应用方式**

1. 修饰实例方法相当于对实例加锁
2. 修饰静态方法相当于类对象加锁
3. 修饰代码块，指定加锁对象，相当于对指定对象加锁

## Synchronzied 底层语义

#### Java对象头和Monitor

对象在内存中的布局 ： 

![img](https://img-blog.csdn.net/20170603163237166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

对象头即为synchronized 的加锁基础，对象头中保存有元数据记录对象状态。synchronized的实现依赖 C++中的`ObjectMonitor` 类 ,这个类结合其三个属性 _WaitSet  和 _EntryList  和 owner 三个属性，共同实现了Object的这也是为什么java中任何对象都可以作为锁，且notify / notifyAll / wait 在Obeject 方法里面且都是native方法



在 java 1.6 之前，synchronized 是重量级锁，效率低。在java 1.6 以后，synchronized 进行了优化。引入了偏向锁和轻量级锁。



### Java 1.6 对 synchronized的 优化

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁 。 随着锁的竞争，锁可以从偏向锁一直升级到重量级锁，且升级是单向的，也就是说锁只能升级无法降级。



#### 偏向锁

偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。

#### 轻量级锁

轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。 

#### 自旋锁

自旋锁的意思是在线程等待锁时，不进行线程切换，因为线程切换涉及到从用户态切换到核心态，这个切换成本较高。自旋就是让线程在等待锁时让他进行循环获取。

#### 锁消除

Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间



### 使用 synctronized 需要注意的点



1. 所谓等待唤醒机制本篇主要指的是notify/notifyAll和wait方法，在使用这3个方法时，必须处于synchronized代码块或者synchronized方法中，否则就会抛出IllegalMonitorStateException异常，这是因为调用这几个方法前必须拿到当前对象的监视器monitor对象，也就是说notify/notifyAll和wait方法依赖于monitor对象
2. `synctronized` 实现的是非公平锁，因为一个锁在对一个对象加锁时会首先尝试加锁，如果失败进入队列，如果成功直接进入。

