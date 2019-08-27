	# NIO

## BIO, NIO,AIO的区别适合的场景

1. BIO 是流导向的，NIO 是缓冲导向的
   1. 在不对数据进行缓存的情况下，你是没办法对数据进行index操作的
   2. 缓冲导向是将数据已经buffer起来的。
2. BIO是阻塞同步IO，NIO是非阻塞同步IO，AIO 是非堵塞异步IO
   1. NIO 相较于AIO而言，NIO是非堵塞的但是 R/W的处理还是需要Process 自己处理，而AIO是交给OS来进行 R/W操作
3. 各自的应用场景 ：
   1. BIO ： 传统的小架构应用，处理少量的应用等
   2. NIO ： 处理连接数量多，且短的应用，如一些websocket应用等等,Reactor 模式
   3. AIO ： 处理大量连接，且IO 压力大的服务器，充分调用OS参与并发，如相册服务器。

## Java BIO,NIO,AIO 底层原理

**NIO 可以通过 selector 使得可以利用一个thread来管理多个channels (网络链接或者files R/W) 但是同样的，需要字节维护好数据的IO操作的完整性。明显的，通过这样的方式，可以节省不少CPU资源，因为如果使用传统BIO的话，每个网络链接或者files R/W 分配一个thread然后处理一个IO操作，这样相对于NIO就能降低不少CPU的压力** 



在UNIX 下的5中IO模型 

1. 阻塞 I/O
2. 非阻塞 I/O
3. I/O 多路复用（select和poll）
4. 信号驱动 I/O（SIGIO）
5. 异步 I/O（Posix.1的aio_系列函数）



## BIO的原理

- 阶段1：等待数据就绪。网络 I/O 的情况就是等待远端数据陆续抵达；磁盘I/O的情况就是等待磁盘数据从磁盘上读取到内核态内存中。
- 阶段2：数据拷贝。出于系统安全，用户态的程序没有权限直接读取内核态内存，因此内核负责把内核态内存中的数据拷贝一份到用户态内存中。



## NIO 原理



NIO selector 的四种状态 ：

1. SelectionKey.OP_CONNECT : Ready to connect 
2. SelectionKey.OP_ACCEPT : Ready to accept 
3. SelectionKey.OP_READ : Ready to read
4. SelectionKey.OP_WRITE : Ready to write



1. **多路复用** ： NIO基于 UNIX下的 `多路复用` IO模型，即一个线程管理多个IO的读写，在NIO中，IO读写的单位不再是线程，而是`channel`。 
2. **零拷贝**  ：  linux下的零拷贝技术，归根到底就是为了避免 IO 拷贝时避免将数据从 内核态写入到用户态然后从用户态写到内核态，通过这样的方式可以避免

#### 多路复用

Java NIO 中的最佳实践，一般都是通过 `Reactor 模型` 实现的，而在Linux服务器中，Reactor的模型一般是通过 poll / epoll 实现([select , poll, epoll之间的区别总结 ](https://www.cnblogs.com/Anker/p/3265058.html))，在从 java SE 6 开始 NIO的selector 默认是通过epoll创建的。**相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。** 

#### 零拷贝技术

对于零拷贝技术([参考](https://juejin.im/entry/59b740fdf265da06633d02cf))，传统的拷贝，进程被动的通过 外部设备发出的中断信号进入到内核态进程磁盘的IO读写，将数据读取到用户态，然后又从用户态将数据写入到内核态，这样就出现了两次数据(如果算上磁盘和内存态的拷 贝，是4次)的拷贝，并且还出现了线程执行指令时出现了内核态和用户态的上下文切换。零拷贝技术通过**[DMA](https://www.jianshu.com/p/870bbbd0f20e) 来完成IO的拷贝，全程不需要CPU参与** 

## NIO的一些最佳实践框架

### Netty

[参考](https://www.quora.com/How-does-a-Netty-server-handle-requests)

[Netty 系列之 Netty 线程模型](https://www.infoq.cn/article/netty-threading-model)

##### 基本原理 

一个独立的 NIO 线程池（Acceptor）接收客户端连接，Acceptor 接收到客户端 TCP 连接请求处理完成后（可能包含接入认证等），将新创建的 SocketChannel 注册到 IO 线程池（sub reactor 线程池）的某个 IO 线程上，由它负责 SocketChannel 的读写和编解码工作。Acceptor 线程池仅仅只用于客户端的登陆、握手和安全认证，一旦链路建立成功，就将链路注册到后端 subReactor 线程池的 IO 线程上，由 IO 线程负责后续的 IO 操作。

![img](https://static001.infoq.cn/resource/image/a4/95/a46b34a7b47249a31b6bcb8a486c9495.png)

##### Netty 中的组件

1. `Selector `:  java IO提供的多路复用器，负责配合操作系统的select / poll 操作票
2. `EventLoop/EventLoopGroup`  ： EventLoopGroup其实就是一个EventLoop线程组，netty中通常有多个EventLoop同时工作，每个EventLoop维护着一个Selector实例（类似单线程Reactor工作）。如果没有显式指定，默认每个EvenLoopGroup中的线程数为可用的`CPU内核数*2`。通常每个netty服务端有两个EventLoopGroup。一个用作Acceptor线程池，负责处理客户端的连接请求，通常一个服务端口对应一个EventLoop线程，根据实际需要配置线程组的线程数量。Acceptor线程通过不断轮询Selector上的Accept事件，将accept的SocketChannel交给另外一个EventLoop线程组。另一个EventLoopGroup会根据线程组的顺序next一个可用的EventLoop将这个SocketChannel注册到其维护的Selector上，并处理其后续的I/O的事
3. ChannelPipleline： 每个SocketChannel都有一个Pipleline实例，而每个Pipleline中维护了一个ChannelHandler链表队列。Pipleline和ChannelHandler的关系类似servlet和filter过滤器的作用。EventLoop从Selector中分离出就绪的channel以后，会将它传递的消息传输到Pipleline中，通过ChannelHandler处理链进行层层处理，用户可以在Handler中添加自己的业务逻辑。

##### netty 的工作步奏

1. 客户端发起请求
2. 由用户线程负责初始化客户端资源，发起连接操作；
3. 如果连接成功，将 SocketChannel 注册到 IO 线程组的 NioEventLoop 线程中，监听读操作位；
4. 如果没有立即连接成功，将 SocketChannel 注册到 IO 线程组的 NioEventLoop 线程中，监听连接操作位；
5. 连接成功之后，修改监听位为 READ，但是不需要切换线程。

##### Reactor 线程 NioEventLoop

一个 NioEventLoop 聚合了一个多路复用器 Selector，因此可以处理成百上千的客户端连接，Netty 的处理策略是每当有一个新的客户端接入，则从 NioEventLoop 线程组中顺序获取一个可用的 NioEventLoop，当到达数组上限之后，重新返回到 0，通过这种方式，可以基本保证各个 NioEventLoop 的负载均衡。一个客户端连接只注册到一个 NioEventLoop 上，这样就避免了多个 IO 线程去并发操作它。



`NioEventLoop` 的工作方式，通过串行的方式执行 ：  

![img](https://static001.infoq.cn/resource/image/c9/49/c9cea26e2e1d13f7b9c2f44b101d3f49.png)

#### Netty 的最佳实践

1. 时间可控的简单业务直接在 IO 线程上处理，如果不需要IO操作，或者等待其他资源，可以直接在`ChannelHanlder` 上进行操作，避免线程的上线文切换
2. 



### Tomcat 

Tomcat 在8.5 以后就完全抛弃了BIO的 connector,因为一些新的 比如 websocket 需要 NIO的支持。另外NIO 在Tomcat 的Keeplive的优化

对于BIO 而言 server 接收 KeepLive 请求，由于 client set了 KeepLive Header , server 在响应了第一次HTTP request 后并不会马上回收为client 分配的 thread， 而NIO不一样，Thread只负责处理 response，请求是acceptor的线程来处理。而NIO不是线程阻塞的，所以在处理时不会肤色acceptor线程。

