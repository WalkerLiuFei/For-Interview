# IO

参考

1. https://www.jianshu.com/p/dfd940e7fca2
2. https://cloud.tencent.com/developer/article/1005481

## IO分类

本质上，select、poll、epoll本质上都是同步I/O，相信大家都读过Richard Stevens的经典书籍UNP（UNIX:registered: Network Programming），书中给出了5种IO模型：

[1] blocking IO - 阻塞IO
[2] nonblocking IO - 非阻塞IO
[3] IO multiplexing - IO多路复用
[4] signal driven IO - 信号驱动IO
[5] asynchronous IO - 异步IO

其中前面4种IO都可以归类为synchronous IO - 同步IO，在介绍select、poll、epoll之前，首先介绍一下这几种IO模型。

### IO-同步，异步，阻塞，非阻塞

一般情况下，一次网络IO读操作会涉及两个系统对象：

(1) 用户进程(线程)Process；

(2)内核对象kernel

两个处理阶段：

```javascript
[1] Waiting for the data to be ready - 等待数据准备好
[2] Copying the data from the kernel to the process - 将数据从内核空间的buffer拷贝到用户空间进程的buffer
```

**IO模型的异同点就是区分在这两个系统对象、两个处理阶段的不同上。**

### Blocking IO

Blocking IO在IO的两个处理阶段都是等待的：

1. 在数据没准备好的时候，process在原地等待kernel 准备数据
2. kernel准备好数据以后，process继续等待kernel将数据读到自己的buffer。

### NonBlocking IO

![img](https://blog-10039692.file.myqcloud.com/1500017008242_1297_1500017008596.png)

和Blocking IO不同，

1. process在询问kernel data是否ready时，如果数据没有ready，kernel会立即响应一个`EWOULDBLOCK`错误，process不会等待data ready，而是做其他工作
2. 当kernel准备好数据后，进入处理的第二阶段的时候，process会等待kernel将数据copy到自己的buffer，在kernel完成数据的copy后process才会从recvfrom系统调用中返回。

### 同步IO 之 IO multiplexing

![img](https://blog-10039692.file.myqcloud.com/1500017024989_2800_1500017025229.png)

​	IO多路复用，就是我们熟知的select、poll、epoll模型。从图上可见，在IO多路复用的时候，process在两个处理阶段都是block住等待的。初看好像IO多路复用没什么用，**select、poll、epoll的优势在于可以以较少的代价来同时监听处理多个IO。**

### 异步IO

![img](https://blog-10039692.file.myqcloud.com/1500017078734_6117_1500017078936.png)

从上图看出，异步IO要求process在recvfrom操作的两个处理阶段上都不能等待，也就是

1. process调用recvfrom后立刻返回，kernel自行去准备好数据并将数据从kernel的buffer中copy到process的buffer在通知process读操作完成了，然后process在去处理。
2. 遗憾的是，linux的网络IO中是不存在异步IO的，linux的网络IO处理的第二阶段总是阻塞等待数据copy完成的。**真正意义上的网络异步IO是Windows下的IOCP（IO完成端口）模型。**

AIO在两个阶段都是非阻塞的！







## Select ，poll,epoll

select ,epoll,poll 其实都是Linux下的多路复用IO，可以监视多个文件描述符，一旦某个描述符就绪（一般指的是读就绪 / 写就绪），能够通知程序进行响应的读写操作。**<font color = red >但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的</font>**

换个说法，其实多路复用IO实在解决IO操作的第一步，也就是等待数据ready的那一步。

### select 

select目前几乎在所有的平台上支持，`其良好跨平台支持也是它的一个优点`。`select的一个缺点在于单个进程能够监视的文件描述符的数量存在最大限制`，在Linux上一般为1024，`可以通过修改宏定义甚至重新编译内核的方式提升这一限制`，但是这样也会造成效率的降低。

`select本质上是通过设置或者检查存放fd标志位的数据结构来进行下一步处理`。这样所带来的缺点是：

1. **select最大的缺陷就是单个进程所打开的FD是有一定限制的，它由FD_SETSIZE设置，默认值是1024。**
2. **对socket进行扫描时是线性扫描，即采用轮询的方法，效率较低。**
3. **需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。**

### Poll

> `poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间`，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。

**它没有最大连接数的限制，原因是它是基于链表来存储的，但是同样有一个缺点：**

> 1. `大量的fd的数组被整体复制于用户态和内核地址空间之间`，而不管这样的复制是不是有意义。
> 2. `poll还有一个特点是“水平触发”`，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

 从上面看，select和poll都需要在返回后，`通过遍历文件描述符来获取已经就绪的socket`。事实上，`同时连接的大量客户端在一时刻可能只有很少的处于就绪状态`，因此随着监视的描述符数量的增长，其效率也会线性下降。

### epoll

epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。`epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次`。

#### 基本原理

`epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次`。还有一个特点是，`epoll使用“事件”的就绪通知方式`，通过epoll_ctl注册fd，`一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd`，epoll_wait便可以收到通知。

#### Epoll 优点

1. `没有最大并发连接的限制`，能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）。
2. `效率提升，不是轮询的方式，不会随着FD数目的增加效率下降`。只有活跃可用的FD才会调用callback函数；`即Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关`，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll。
3. `内存拷贝`，利用mmap()文件映射内存加速与内核空间的消息传递；`即epoll使用mmap减少复制开销`。



## Select， Poll ， epoll的区别

### 最大连接数限制

| select | 单个进程能打开的最大连接数有限，32位机器(1024),64位机器(2048) |
| ------ | ------------------------------------------------------------ |
| poll   | poll使用链表存储fds，所以没有最大连接数限制                  |
| epoll  | 由最大连接数限制，但是是受限于内存的                         |

### FD剧增后带来的IO效率问题

| select | 因为每次调用时都会对连接进行线性遍历，所以随着FD的增加会造成遍历速度慢的"线性下降性能问题" |
| ------ | ------------------------------------------------------------ |
| poll   | 同上                                                         |
| epoll  | 因为epoll是内核中实现是根据每个fd上面的callback函数来实现的，只有活跃的socket才会主动callback，所以在活跃socket较少的情况下，使用epoll没有前面两者的性能下降问题 |

### 消息传递方式

| select | 内核需要将消息传递到用户空间，需要内核拷贝工作         |
| ------ | ------------------------------------------------------ |
| poll   | 同上                                                   |
| epoll  | epoll通过内核和用户空间共享一块内存来实现的 （事件表） |



