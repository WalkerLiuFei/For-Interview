# Nginx 原理

## Nginx的模块和工作原理

Nginx的模块从结构上分为核心模块、基础模块和第三方模块：

+ **核心模块**：HTTP模块、EVENT模块和MAIL模块

+ **基础模块**：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块，

+ **第三方模块**：HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块。

用户根据自己的需要开发的模块都属于第三方模块。正是有了这么多模块的支撑，Nginx的功能才会如此强大。


Nginx的模块从功能上分为如下三类。

+ **Handlers（处理器模块）**。此类模块直接处理请求，并进行输出内容和修改headers信息等操作。Handlers处理器模块一般只能有一个。

+ **Filters （过滤器模块）** 。此类模块主要对其他处理器模块输出的内容进行修改操作，最后由Nginx输出。

+ **Proxies （代理类模块）**。此类模块是Nginx的HTTP Upstream之类的模块，这些模块主要与后端一些服务比如FastCGI等进行交互，实现服务代理和负载均衡等功能。

![img](https://img-blog.csdn.net/20130515152325076) 

Nginx本身做的工作实际很少，当它接到一个HTTP请求时，它仅仅是通过查找配置文件将此次请求映射到一个location block，而此location中所配置的各个指令则会启动不同的模块去完成工作，因此模块可以看做Nginx真正的劳动工作者。通常一个location中的指令会涉及一个handler模块和多个filter模块（当然，多个location可以复用同一个模块）。handler模块负责处理请求，完成响应内容的生成，而filter模块对响应内容进行处理。

## Nginx的进程模型

Nginx分为**单工作进程**和**多工作进程**两种模式。

1. 单工作进程 : 除主进程外，还有一个工作进程，工作进程是单线程的；
2.  多工作进程 : 每个工作进程包含多个线程。Nginx默认为单工作进程模式。

Nginx在启动后，会有一个master进程和多个worker进程。

### 多进程工作模式

1. Nginx 在启动后，会有一个 master 进程和多个相互独立的 worker 进程。
2. 接收来自外界的信号，向各worker进程发送信号，每个进程都有可能来处理这个连接。(通过争抢 `accept_mutex`来实现 )
3.  master 进程能监控 worker 进程的运行状态，当 worker 进程退出后(异常情况下)，会自动启动新的 worker 进程

### master进程

​	master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。

​	master进程充当整个进程组与用户的交互接口，同时对进程进行监护。它不需要处理网络事件，不负责业务的执行，只会通过管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。

#### worker 职责

1. 读取并验证配置信息。 
2. 创建，绑定，关闭套接字。
3. 启动，终止，维护worker进程的个数。
4. 通过 `kill HUP ` 或者 `nginx -s reload` 向Nginx Master进程 发送 signal信息重新加载配置文件 : **如果主配置文件发生改变，那么并不会立刻影响到WORKER进程，而是MASTER等到WORKER进程的连接请求处理完毕后KILL掉这个WORKER进程，然后重新生成一个WORKER进程，这样这个WORKER进程就将以新的配置启动了。也就是说，老的连接用老的配置，新的连接用新的配置。重新加载配置文件不会中断正在处理的请求。**

### Work进程

​	而基本的网络事件，则是放在worker进程中来处理了。多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。**一个请求，只可能在一个worker进程中处理，一个worker进程，不可能处理其它进程的请求。** **worker进程的个数是可以设置的，一般我们会设置与机器cpu核数一致，这里面的原因与nginx的进程模型以及事件处理模型是分不开的。**

worker进程之间是平等的，每个进程，处理请求的机会也是一样的。当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接，怎么做到的呢？

1. **master进程fork 出worker 进程 : ** 进程从master进程 fork过来,在master进程里面，先建立好需要listen的socket（listenfd）之后，然后再fork出多个worker进程。
2. **所有worker进程的`listenfd`会在新连接到来时变得可读**，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢`accept_mutex`，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。
3. 当一个worker进程在accept这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接.
4. **Nginx采用了异步非阻塞事件驱动的方式来处理请求的，`只要我们设置好WORKER进程个数与CPU的亲缘性绑定`**，那么就能减少CPU在进程间切换所花费的时间以及切换带来的进程的保存/恢复现场，同时，由于Nginx中一个worker里面只有一个线程，也避免了线程的上下文切换。



![img](https://img-blog.csdn.net/20160401103647146)

### Nginx + FastCGI

​	FastCGI接口方式采用C/S结构，可以将HTTP服务器和脚本解析服务器分开，同时在脚本解析服务器上启动一个或者多个脚本解析守护进程。当HTTP服务器每次遇到动态程序时，可以将其直接交付给FastCGI进程来执行，然后将得到的结果返回给浏览器。这种方式可以让HTTP服务器专一地处理静态请求或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。 

必须通过FastCGI接口来调用。FastCGI接口在Linux下是**socket**

wrapper：为了调用CGI程序，还需要一个FastCGI的wrapper（wrapper可以理解为用于启动另一个程序的程序），这个wrapper绑定在某个固定socket上，如端口或者文件socket。当Nginx将CGI请求发送给这个socket的时候，通过FastCGI接口，wrapper接收到请求，然后Fork(派生）出一个新的线程，这个线程调用解释器或者外部程序处理脚本并读取返回数据；接着，wrapper再将返回的数据通过FastCGI接口，沿着固定的socket传递给Nginx；最后，Nginx将返回的数据（html页面或者图片）发送给客户端。这就是Nginx+FastCGI的整个运作过程，如图1-3所示。


![img](https://img-blog.csdn.net/20130516093049837) 

所以，我们首先需要一个wrapper，这个wrapper需要完成的工作：

1. 通过调用fastcgi（库）的函数通过socket和ningx通信（读写socket是fastcgi内部实现的功能，对wrapper是非透明的）
2. 调度thread，进行fork和kill
3. 和application（php）进行通信






​    

