[参考](http://veithen.io/2014/01/01/how-tcp-backlog-works-in-linux.html)

当应用程序使用监听系统调用将套接字置于LISTEN状态时，它需要为该套接字指定一个积压(backlog)。

`backlog`通常被描述为传入连接队列的限制。 

由于TCP使用三向握手，因此传入连接在到达`ESTABLISHED`状态之前会经过`SYN` `RECEIVED`中间状态，并且可以通过`accept` `syscall`返回到应用程序。 这意味着TCP / IP堆栈具有两个选项来为处于LISTEN状态的套接字实现积压队列：

1. 通过一个单队列其size由系统调用`listen`时的参数 `backlog`决定。收到SYN数据包后，它将发回SYN / ACK数据包，并将连接添加到该队列中。接收到相应的ACK后，连接会将其状态更改为ESTABLISHED，并有资格切换到应用程序。这意味着队列可以包含处于两种不同状态的连接：SYN RECEIVED和ESTABLISHED。 accept syscall只能将后一种状态的连接返回给应用程序。
2. 该实现使用两个队列，一个`SYN`队列（或不完整的连接队列）和一个接受队列（或完整的连接队列）。状态为`SYN` ,`RECEIVED`的连接被添加到`SYN`队列中，然后在它们的状态更改为`ESTABLISHED`时移至接受队列.顾名思义，然后简单地实现accept调用以消耗来自accept队列的连接。在这种情况下，listen syscall的backlog参数确定接受队列的大小。

BSD 实现TCP通过第一种方式，选择第一种方式意味着，当连接数量大道 backlog数量后 ，系统不会再响应对应 `SYNC/ACK` 消息取响应 client的`SYNC`消息 。通常，TCP实现将简单地丢弃SYN数据包（**而不是使用RST数据包进行响应），以便客户端重试。** 

请注意，Stevens实际上解释了BSD实现确实使用了两个单独的队列，但是它们表现为单个队列，其最大大小由backlog参数确定（但不必完全等于backlog 参数）



**当前的Linux版本将第二个选项与两个不同的队列一起使用：一个SYN队列，其大小由系统范围的设置指定；一个接受队列，其大小由应用程序指定**

1. sync queue : 未建立的 TCP连接队列，大小由系统设定
2. accept queue  ： 已经建立的TCP队列，大小由应用设定



**那么问题来了： 如果 `accept queue` 已经满了，`sync queue` 里面的已经建立的`connection` 已经无法移到`accept queue` 中 怎么办？ 深入源码可以了解到。 需要将`/proc/sys/net/ipv4/tcp_abort_on_overflow` 这个值设置为 1， 通过这种方式，系统在处理时会为client发送一个 `RESET` packet， 方便client重新发起，重试的算法时 [指数退避算法](https://en.wikipedia.org/wiki/Exponential_backoff)**  其指数系数在`/proc/sys/net/ipv4/tcp_synack_retries` 里面。

如果不设置这个值，系统就会将新进来的connection discard 掉！



## 半连接状态的产生

首先回顾三次握手的机制 ：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwNjA3MjA1NzA5MzY3?x-oss-process=image/format,png)

另一方面，如果客户端首先等待来自服务器的数据并且服务器从不减少积压，则最终结果是在客户端，连接处于已建立状态，**而在服务器端，该连接被视为已关闭。这意味着我们最终将获得半开放式连接！**



解决方案只是增加积压。系统管理员可以根据流量特征调整`/ proc / sys / net / ipv4 / tcp_max_syn_backlog。` 