# Java NIO 和servlet 3.0

[async-servlet-example](https://www.journaldev.com/2008/async-servlet-example)

[java-concurrency-asynchronous-processing-support-in-servlet-3-0](https://www.javaworld.com/article/2077995/java-concurrency-asynchronous-processing-support-in-servlet-3-0.html)

[spring boot--使用异步请求，提高系统的吞吐量](https://blog.csdn.net/liuchuanhong1/article/details/78744138)  

[异步请求 benchmark](https://www.jianshu.com/p/7e5290223246)

[is-servlet-3-1-readwritelistener-supported-by-deferredresult-in-spring-4](https://stackoverflow.com/questions/28828355/is-servlet-3-1-readwritelistener-supported-by-deferredresult-in-spring-4)

## Servlet 3.0以前

对于每个请求，servlet 都分配一个线程去处理，直到最后的响应。这样会导致`Thread Starvation ` 问题。即，如果大量的请求进入到我们的server，那么由于每个请求都分配一个Thread去处理，如果已分配的线程达到了我们的limit上限，那么就会造成`Thread Starvation` 问题。

 在Servlet 3.0之前，对于这些长时间运行的线程，存在容器特定的解决方案，我们可以生成一个单独的工作线程来执行繁重的任务，然后将响应返回给客户端。 启动工作线程后，servlet线程返回到servlet池。 Tomcat的Comet，WebLogic的FutureResponseServlet和WebSphere的异步请求调度程序是异步处理实现的一些示例。

## Servlet 3.0 Async Processing



<img src="https://static.oschina.net/uploads/img/201710/24080523_8BkS.png" alt="tomcatæ¶æå¾.png" style="zoom:50%;" />

由上图可知，当接收到request请求之后，由tomcat工作线程从HttpServletRequest中获得一个异步上下文AsyncContext对象，然后由tomcat工作线程把AsyncContext对象传递给业务处理线程，同时tomcat工作线程归还到工作线程池，这一步就是异步开始。在业务处理线程中完成业务逻辑的处理，生成response返回给客户端。在Servlet3.0中虽然处理请求可以实现异步，但是InputStream和OutputStream的IO操作还是阻塞的，当数据量大的request body 或者 response body的时候，就会导致不必要的等待。从Servlet3.1以后增加了非阻塞IO，需要tomcat8.x支持。



Servlet 3.0 将接收请求的HTTP Thread 和为请求分配的Work Thread 解耦。请求大量耗时工作交给Work Thread 处理，HTTP Thread 只负责接受请求。

AsyncContext的目的并不是为了**提高性能**，也并不直接提供性能提升，它提供了把HTTP thread和Worker thread解藕的机制，从而提高Web容器的**响应能力**。

**一定不要认为Worker thread pool必须比HTTP thread pool大**，理由如下：

1. 两者职责不同，一个是Web容器用来接收外来请求，一个是处理业务逻辑。
2. thread的创建是有代价的，如果HTTP thread pool已经很大了再搞一个更大的Worker thread pool反而会造成过多的Context switch和内存开销。
3. AsyncContext的目的是将HTTP thread释放出来，避免被操作长期占用进而导致Web容器无法响应。

### Spring MVC 是怎么使用Async Processing的

Spring MVC执行异步操作需要用到`AsyncTaskExecutor` ，（Spring Boot 默认开启，也就是@EnableAsync 注解）。



利用Callable接口，一个响应callable 

```java
@RestController
public class CallableController {

  @RequestMapping("callable-hello")
  public Callable<String> hello() {
    return () -> new SlowJob("CallableController").doWork();
  }
}
```

 容器的线程http-nio-8060-exec-1这个线程进入controller之后，就立即返回了，具体的服务调用是通过另一个线程（work thread） 来做的，当服务执行完要返回后，容器会再启一个新的线程http-nio-8060-exec-2来将结果返回给客户端或浏览器，整个过程response都是打开的，当有返回的时候，再从server端推到response中去。 

相似的还有 :

1. 利用`WebAsyncTask` 来进行，这种方式和上面的`callable`方式最大的区别就是，WebAsyncTask支持超时，并且还提供了两个回调函数，分别是`onCompletion`和`onTimeout`，顾名思义，这两个回调函数分别在执行完成和超时的时候回调。
2.  `DeferredResult`,用于解决 和第三方唤起 / 等待数据等等 有关的请求，例如JMS，定时任务，队列等。



##  Servlet 3.1 Async IO

**In other words, with Servlet 3.0, only the request processing part became async, but not the I/O for serving the requests and responses. If enough threads block, this results in thread starvation and affects performance.**

servlet3.1 通过	`ReadListener` 和 `WriteListener`接口解决这种问题，These are registered in `ServletInputStream` and `ServletOutputStream`. The listeners have callback methods that are invoked when the content is available to be read or can be written without the servlet container blocking on the I/O threads. So these I/O threads are freed up and can now serve other request increasing performance. 

### Spring 怎么使用 Async IO的

Spring 使用async IO 受限于具体的容器实现。