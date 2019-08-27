# Spring 事务

## @Transactional vs Spring TransactionTemplate

### 声明式事务的实现方式

声明式事务通过AOP实现，代码入侵小。



![tx](https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/images/tx.png) 





## Spring 事务的优势

### 全局事务 （分布式事务）

全局事务Spring通过消息队列实现，原始的应用来解决分布式事务通过JTA实现，非常繁琐，此外使用JTA还需要JNDI。



## Spring 分布式事务的实现



## Spring 事务的传播属性



事务的传播属性 `PROPAGATION` 是Spring框架独有的事务增强特性，它不属于事务实际提供方数据库行为。

| 事务传播行为类型            |                             说明                             |
| --------------------------- | :----------------------------------------------------------: |
| `PROPAGATION_REQUIRED`      | 默认的传播属性，支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是默认的事务管理方式 |
| `PROPAGATION_SUPPORT`       |     支持当前事务，如果当前没有事务，就以非事务方式执行。     |
| `PROPAGATION_MANDATORY`     |        使用当前的事务，如果当前没有事务，就抛出异常。        |
| `PROPAGATION_REQUIRED_NEW`  |          新建事务，如果当前存在事务，把当前事务挂起          |
| `PROPAGATION_NOT_SUPPORTED` |      以非事务方式执行操作，如果当前存在事务，把事务挂起      |
| `PROPAGATION_NEVER`         |       以非事务方式执行，如果当前存在事务，则抛出异常。       |
| `PROPAGATION_NESTED`        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |



## 事务的隔离级别

1.  ISOLATION_DEFAULT： 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别. 另外四个与JDBC的隔离级别相对应

  2. ISOLATION_READ_UNCOMMITTED： 这是事务最低的隔离级别，它充许令外一个事务可以看到这个事务未提交的数据。 这种隔离级别会产生脏读，不可重复读和幻像读。
 3. ISOLATION_READ_COMMITTED： 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据
 4. ISOLATION_REPEATABLE_READ： 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。
      它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。
 5. ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。
      除了防止脏读，不可重复读外，还避免了幻像读。

 