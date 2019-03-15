# 集合spring框架面试真题——springmvc、springcloud、springboot



## **1.Spring**

Spring 框架现在是 Java 后端框架家族里面最强大的一个，其拥有 IOC 和 AOP 两大利器，大大简化了软件开发复杂性。并且，Spring 现在能与所有主流开发框架集成，可谓是一个万能框架，Spring 让 JAVA 开发变得更多简单。
**面试真题：**（1）什么是控制反转（IOC）？什么是依赖注入？
（2）请解释下Spring中的IOC？
（3）将Spring配置到你的应用中共有几种方法？
（4）怎样用注解的方式配置Spring？
（5）描述Spring Bean的生命周期？
（6）描述Spring中各种Bean的范围？
（7）什么是Spring的嵌入beans？
（8）Spring框架中的单例bean是否是线程安全的？
（9）请举例说明如何用Spring注入一个Java的集合类？
（10）请举例说明如何在Spring的Bean中注入一个java.util.Properties？
（11）请解释Spring的Bean的自动生成原理？
（12）请辨析自动生成Bean之间模块的区别？
（13）如何开启基于基于注解的自动写入？
（14）请举例说明@Required注解？
（15）请举例说明@Autowired注解？
（16）请举例说明@Qualifier注解？
（17）请说明构造器注入和setter方法注入之间的区别？
（18）Spring框架中不同类型event有什么区别？
（19）FileSystemResource和ClassPathResource有何区别？
（20）请列举Spring框架中用了哪些设计模式？

## **2.Spring MVC**

Spring MVC 是一个 MVC 开源框架，用来代替 Struts。它是 Spring 项目里面的一个重要组成部分，能与 Spring IOC 容器紧密结合，以及拥有松耦合、方便配置、代码分离等特点，让 JAVA 程序员开发 WEB 项目变得更加容易。
**面试真题：**（1）Spring MVC和Struts2的异同 ？
（2） Spring MVC的异常处理 ？
（3） SpringMVC中的拦截器问题 ？
（4）Spring MVC如何解决中文乱码问题 ？
（5） 系统如何分层 ？
（6） Spring MVC如何向页面传值 ？
（7）SpringMVC如何读取请求参数值 ？
（8）基于注解的Spring MVC的应用编程步骤
（9）Spring MVC的优点：
（10）SpringMVC怎样设定重定向和转发 ？
（11）SpringMvc的控制器是不是单例模式,如果是,有什么问题,怎么解决
（12）SpingMvc中的控制器的注解一般用那个,有没有别的注解可以替代
（13）@RequestMapping注解用在类上面有什么作用
（14）怎么样把某个请求映射到特定的方法上面
（15） 如果在拦截请求中,我想拦截get方式提交的方法,怎么配置
（16） 如果在拦截请求中,我想拦截提交参数中包含”type=test”字符串,怎么配置
（17）我想在拦截的方法里面得到从前台传入的参数,怎么得到
（18） 如果前台有很多个参数传入,并且这些参数都是一个对象的,那么怎么样快速得到这个对象
（19） 怎么样在方法里面得到Request,或者Session
（20） SpringMvc中函数的返回值是什么.

## **3.Spring Boot**

Spring Boot 是 Spring 开源组织下的一个子项目，也是 Spring 组件一站式解决方案，主要是为了简化使用 Spring 框架的难度，简省繁重的配置。Spring Boot提供了各种组件的启动器（starters），开发者只要能配置好对应组件参数，Spring Boot 就会自动配置，让开发者能快速搭建依赖于 Spring 组件的 Java 项目。
**面试真题：**（1）Spring Boot 还提供了其它的哪些 Starter Project Options？
（2）为什么要用 Spring Boot？
（3）Spring Boot 的核心配置文件有哪几个？它们的区别是什么？
（4）Spring Boot 的配置文件有哪几种格式？它们有什么区别？
（5）Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？
（6）开启 Spring Boot 特性有哪几种方式？
（7）Spring Boot 需要独立的容器运行吗？
（8）运行 Spring Boot 有哪几种方式？
（9）Spring Boot 自动配置原理是什么？
（10）Spring Boot 的目录结构是怎样的？
（11）你如何理解 Spring Boot 中的 Starters？
（12）如何在 Spring Boot 启动的时候运行一些特定的代码？
（13）Spring Boot 有哪几种读取配置的方式？
（14）Spring Boot 支持哪些日志框架？推荐和默认的日志框架是哪个？
（15）SpringBoot 实现热部署有哪几种方式？
（16）你如何理解 Spring Boot 配置加载顺序？
（17）Spring Boot 如何定义多套不同环境配置？
（18）Spring Boot 可以兼容老 Spring 项目吗，如何做？
（19）保护 Spring Boot 应用有哪些方法？
（20）Spring Boot 2.X 有什么新特性？与 1.X 有什么区别？

## **4.Spring Cloud**

Spring Cloud 是一系列框架的有序集合，是目前最火热的微服务框架首选，它利用Spring Boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 Spring Boot 的开发风格做到一键启动和部署。
**面试真题：**（1）选择SpringCloud作为微服务架构的原因
（2）SpringBoot和SpirngCloud，请你谈谈对他们的理解
（3）什么是服务熔断？什么是服务降级？
（4）微服务的优缺点分别是什么？
（5）你所知道的微服务技术栈有哪些？
（6）springcloud断路器的作用
（7）什么是微服务
（8）springcloud如何实现服务的注册和发现
（9）ribbon和feign区别
（10）springcloud断路器的作用
（11）说下你在项目开发中碰到的坑
（12）微服务技术栈有哪些?