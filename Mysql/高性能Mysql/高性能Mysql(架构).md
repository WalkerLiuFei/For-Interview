

## MYSQL架构

1. **事务ACID指的都是那些特性**

2. **MYSQL的并发控制分为几个层面？那个层面的优先度高？**

3. **MYSQL锁的类型有哪些？**

4. **事务的隔离级别有哪些？ 都有那些特性？哪一个是InnoDB的默认隔离级别？**

5. **什么是脏读，幻读，可重复读，串行读？分别都有什么问题存在？**

6. **模拟一个死锁的查询,InnoDB是怎样处理死锁的？**

7. **InnoDB的MVVC(多版本并发控制)是怎样实现的?它可以解决什么问题？带来的利好和弊端都有哪些？**

8. MVVC增删改查时时怎样依赖/操作版本号的？

9. 乐观并发控制和悲观并发控制

10. 什么是幻读，RC(Read Committed) 为什么也会出现幻读

    

## 以上问题

1. A：atomic  原子性 ， C：consistency 一致性  I：隔离性 isolation  D: durability 持久性

2. Mysql的并发控制分为 服务器层面和存储引擎层面，服务器层面你的优先级更高，且锁的力度更大

3. Mysql 的锁：按照隔离级别分为读写锁，其中读锁是共享锁，写锁是排他锁。按照锁的力度分为 表锁和行级锁，其中 Innodb已经支持行级锁

4. 事务的隔离级别

   1. 未提交读（uncommit read）： 事务中的修改即使没有提交，对其他事务也是可见的，即事务可以读取未提交的数据。这会导致**脏读的问题**
   
   2. 提交读（commit read）：也叫 不可重复读 ，  一个事务开始之后，只能读取自己事务内所做的修改。未提交之前，事务内的修改对其他事务是不可见的。 **这会导致事务中 一个查询在连续两次查询中会得到不同的结果，因为可能查询的事务已经处理完**
   
   3. 重复读（REPEATBLE READ） ，又称幻读。该级别解决了提交读的不可重复读的问题。即：在同一个事务中一个查询连续两次查询同一条数据的，结果是一致的。但是这又会导致一个**幻读**的问题，即事务内读到的数据可能已经被做了修改，读到的数据是其他事务做过修改的数据，而不是最新的数据。 z这个隔离级别是Mysql的默认隔离级别。 对于幻读，MYSQL中的**InnoDB 利用MVCC（并发版本控制）来解决**
   
   4. 可串行读（SERILIZABLE READ）, 解决了以上的所有的问题，但是效率低，在高数据一致性的应用上会使用，因为效率实在太低
5. 脏读，幻读，可重复读，串行读
   1. 脏读： 事务读到未提交数据  。
   2. 不可重复读： 事务内连续读同一条数据，得到不同结果 
   3. **幻读：具体一点 ，事务中某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：select 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。**

6.  死锁 ： 两个事务在同一资源上相互占用， 对于死锁的处理，InnoDB目前的处理方式是，将持有最少行级排它锁的事务进行回滚。比如两个输入同时操作两条数据，但是这两个事务在操作这两条数据时的顺序是相反的。InnoDB 实现了锁超时机制和死锁检测机制。另外对于检测到死锁时，InnoDB会将持有最少行级排它锁的事务回滚。

7. **MVVC**
   1. MVVC 是行级锁的一个变种，MVCC可以有很多情况下避免加锁操作。读操作可以避免加锁操作，写操作也只锁定必要的行。
   2. MVVC 通过对具体的Row 进行快照，这样，可以保证一个事务在启动，不论执行多长时间，它之后对一个Row读取到的信息都是一样的。但是根据每个事务开始时间的不同，读取到的内容也有所不同。
   3. MVVC 通过每行加上两个隐式行，一个保存行的创建时间，一个是过期时间。MVCC 只能够在REPEATABLE READ 和 READ COMMITED两个级别下工作
   4. 
8. 





5. [什么是幻读](<https://segmentfault.com/a/1190000016566788)