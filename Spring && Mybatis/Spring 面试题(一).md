## Spring面试题



1. Spring AOP与IOC的实现原理
   1. AI
2. Spring的`BeanFactory`和`FactoryBean`的区别
   1. 
3. **为什么CGlib方式可以对接口实现代理？** 
   1. CGLIB通过为代理类创建一个继承的子类，用子类代理父类，顺势置入切面逻辑，这也是Spring AOP的原理之一。
   2. CGLib采用的是用创建一个继承实现类的子类，用asm库动态修改子类的代码来实现的，所以可以用传入的类引用执行代理类。
   3.  CGLib创建的动态代理对象性能比JDK创建的动态代理对象的性能高不少，但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理。
4. **RMI与代理模式**
5. **Spring的事务隔离级别，实现原理**
6. **对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的**
7. **Mybatis的底层实现原理** 
   1. 
8. MVC框架原理，他们都是怎么做url路由的
   9. 以Method为单位，将RquestMapping的值和Method在Map中做了映射
   2. Request下达到DispatcherSerlet后，通过Map映射就可以找到对应的处理请求处理方法
   3. URL路径匹配类似于正则的表达式
9. 