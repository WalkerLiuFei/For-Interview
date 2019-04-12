# Spring

## Spring MVC原理

### 整体流程

![img](https://img-blog.csdn.net/20141129165243297?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhb2xpamluZzIwMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

具体步骤：

```
   1、  首先用户发送请求——> DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

   2、  DispatcherServlet——> HandlerMapping，HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一个 Handler 处理器（页面控制器）对象、多个 HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；

   3、  DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；

   4、  HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个 ModelAndView 对象（包含模型数据、逻辑视图名）；

   5、  ModelAndView 的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的 View，通过这种策略模式，很容易更换其他视图技术；

   6、  View——>渲染，View 会根据传进来的 Model 模型数据进行渲染，此处的 Model 实际是一个 Map 数据结构，因此很容易支持其他视图技术；

   7、  返回控制权给 DispatcherServlet，由 DispatcherServlet 返回响应给用户，到此一个流程结束。
```

1. DispatcherServerlet  实现了 Servlet , 真正的 服务在 service 方法中。像Tomcat 是一个Servlet Container .

   对于 Tomcat 的研究  : [可以本地调试的Tomcat](<https://github.com/WalkerLiuFei/Tomcat9.0_SourceCode_WithIdea>)

   几篇有关于Tomcat的文章 ：

   1. [整体架构](https://www.jianshu.com/p/3ede935ce8fe>)
   2. [启动](https://www.jianshu.com/p/a26ea654f69e)
   3. [Connector](https://www.jianshu.com/p/f007b6427d60)

2. DispatcherServlet 为`GET`,`HEAD` 请求提供了缓存功能 ，header  中 last-modified 请求头来标识

3. `HandlerExecutionChain` 负责对请求进行处理，里面包含有 InterceptorList 拦截器等，可以对请求进行preHandler,postHandler 处理等。


## Spring MVC 整个处理流程

从进入`doService` 方法以后，进入`doDispatch`  后通过通过uri 匹配到 RequestMapping 注解的Mapping，此Mapping 为一个 注解的对象

1. **所有RequestMapping 注解的方法都会生成 RequestMapingInfo，然后存在RequestMappingHandlerMapping 对象中，以uri 为key，以 MappingRegistration 作为value，里面包含了RequestMapping的信息 和 handlerMethod的反射信息**

2. 在请求进入以后会通过uri 找到对应的method 进行处理。

3. 最终唤起是在 `RequestMappingHandlerAdapter` 中的invokeHandlerMethod 方法中对其进行唤起的。




