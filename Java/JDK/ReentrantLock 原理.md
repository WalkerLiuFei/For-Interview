# ReetrantLock原理

ReentrantLock是基于AQS实现的，这在下面会讲到，AQS的基础又是CAS.

ReetrantLock 通过对

## AQS 原理 

AQS 也就是`AbstractQueuedSynchronizer` 的简称, 是`J.U.C`的核心类.`CountDownLatch、FutureTask、Semaphore` 等都是依赖它实现的.**AQS是基于FIFO队列的实现**，

### state 

`state` 是AQS队列目前的状态 ,int 类型 `volatile` 修饰. `state` 的含义由具体的实现类来进行定义,比如说在`ReetrantLock`里面 当`state == 0` 时 锁的状态就是未加锁的, `ReetrantLock`通过`tryLock` 获取锁时 state 会加1 (CAS操作), 当Release 时 `state` 值减1 直到这个值为0时锁被释放. 这个 state的状态改变只能通过   `getState`, `setState` and `compareAndSetState` 三个方法进行.





### FIFO 队列和Node

FIFO 队列维护了一个双向链表 ,链表中维护的是等待`state`的所有的线程信息,这个链表中的值为Node `J.U.C.AQS$Node` 他是AQS的一个内部类.

#### Share 和 Exclusive

`Node`支持两种状态,一种是共享,一种是排它锁 .如果Node处在共享状态,它共享的对象其实是一个双向链表(同样是Node). 

打个比方,`ReadWriteLock`的实现, 因为写锁是排他锁. `Read`是共享的. 当一个Thread获取到写锁以后,后续的Read操作的Thread其实是可以共享的.

#### 公平锁

不同于`synchronized` ,ReetrantLock 可以通过`FIFO`链表实现公平锁,即先来先得获取锁.



### CAS

