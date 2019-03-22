# 滴水穿石（Mysql知识）

1. MYISAM 引擎和INNODB引擎 各自的适用业务场景都有那些：
   + MYISAM不支持事务，所以它适用的场景集中在检索领域
     + MyISAM可被压缩，存储空间较小InnoDB的表需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引
   + MYISAM不支持事务，InnoDB支持
   + MYISAM锁的级别为表锁，InnoDB是行级别的锁,**InnoDB通过一个聚合索引对行数据进行加锁，行级锁通[聚合索引](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)对数据行进行锁定，聚合索引一般是建立在主键上的，所以可以保证数据唯一性**
   + count without where 时，也就是对表进行统计时，MYISAM的性能更优，因为MYISAM将信息保存到了表中，而InnoDB需要进行全表的扫描

**综上，我个人的总结： 在进行读写分离时，可以将读的库的存储引擎设置为MYISAM，将写的库设置为InnoDB**

2.  [为什么这几个SQL查询的结果集不同](https://stackoverflow.com/questions/5658457/not-equal-operator-on-null)

  +  <>和 != 是对值进行的判定，但是NULL不是一种“值”，NULL的含义是指的这个列所指定的行值是不是为空存放的空间是不是为空。所以在平时判定一个列的null值是一般是用 is null 或者 is not null
```SQL
  SELECT * FROM MyTable WHERE MyColumn != NULL (0 Results)
  SELECT * FROM MyTable WHERE MyColumn <> NULL (0 Results)
  SELECT * FROM MyTable WHERE MyColumn IS NOT NULL (568 Results)
```
3. 查询时是否使用了索引和查询的时候是否扫描全表并不是完全相关联的，查询条件中使用了不等于操作符（<>、!=）的select语句执行慢解决方法：通过把不等于操作符改成or，可以使用索引，避免全表扫描。例如，把column<>’aaa’，改成column<’aaa’ or column>’aaa’，就可以使用索引了。

4. [会导致全表扫描的几种情况](http://www.cnblogs.com/feiling/p/3393356.html)

5. [利用profiling 对sql语句进行解剖](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)
    1. 利用set profiling = 1 打开开关
    2. 利用 SHOW PROFILES 查看语句的query_id 
    3. 利用下面的语句对sql语句进行分析

在ICP之前，Mysql在复合索引中，第一列是范围查询，第二列通常是无法使用索引的，建议第一列是=,<=>,is null。而在ICP出来之后，就没有了这个限制。

```
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]

type:
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS
```

不要将table的主键设置的过长，如果过长的话会导致二级索引所占磁盘空间很大

6. 事物的传播行为: [参考文档](https://blog.csdn.net/it_wangxiangpan/article/details/24180085)
7. ​

