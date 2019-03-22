# Mysql官方文档阅读笔记（Select 语句优化）



[官方文档目录](https://dev.mysql.com/doc/refman/5.7/en/statement-optimization.html)

## 优化select语句
> [原文](https://dev.mysql.com/doc/refman/5.7/en/select-optimization.html)

查询语句作为最常用类型的语句，其优化价值理所当然是最大的。

优化查询语句的最主要的点：

1. 最优先的考虑是是否可以添加索引，在Where的条件列上添加索引，一般是可以大幅度提高查询效率的。不过，索引也是需要占用一定存储空间的，所以，建立高效的索引是关键。可以参考：[mysql是怎样使用索引的](https://dev.mysql.com/doc/refman/5.7/en/mysql-indexes.html)和[使用explain来优化查询](https://dev.mysql.com/doc/refman/5.7/en/using-explain.html)
2. 隔离和调整查询的任何部分，例如函数调用，这需要花费很多时间。根据查询的结构方式，可以为结果集中的每一行调用一次函数，或者甚至对表中的每一行调用一次函数，从而大大放大了任何低效率。
3. 尽量避免进行全表扫描的查询。
4. 有周期的使用[ANALYZE_TABLE](https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html)语句对表进行分析，这样优化器就有构建高效执行计划所需的信息
5. 对存储引擎进行特定优化[InnoDB优化](https://dev.mysql.com/doc/refman/5.7/en/optimizing-innodb-queries.html)
6. 您可以InnoDB使用第8.5.3节[优化InnoDB只读事务](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-ro-txn.html)中的技术来优化表的 单查询事务 。
7. 避免以难以理解的方式转换查询，特别是如果优化程序自动执行某些相同的转换。
8. 如果性能问题不能通过其中一个基本准则解决，请通过阅读EXPLAIN计划并调整索引，WHERE子句，连接条款等来调查特定查询的内部详细信息。**对于平时工作来说，前期项目赶得急，为了项目进度语句可以写的随意一些，后期优化sql是少不了的**
9. 调整MySQL用于缓存的内存区域的大小和属性。通过有效利用 InnoDB 缓冲池， MyISAM密钥缓存和MySQL查询缓存，重复的查询运行速度更快，因为在第二次和随后的时间内从内存中检索结果。
10. 增加缓存的利用效率，一般来说，只要缓存可以生效，查询的效率基本就不会是问题
11. 处理锁定问题，您的查询速度可能会受到其他会话同时访问表的影响。

### 对Where子语句优化 

1. 索引使用的常量表达式只使用一次
2. 在MYISAM存储引擎的表中，使用count(*)并且没有where查询条件下，count(*)是直接从表的信息中获取的结果
3. 如果有一个ORDER BY子句和一个不同的GROUP BY子句，或者如果 ORDER BY或者GROUP BY 包含来自连接队列中的第一个表之外的表的列，则创建一个临时表。

### 查询范围优化

##### 对单个索引查询范围的优化

对于不同类型的索引单个索引，其在where条件中有效的查询范围包括

| hash索引                                   | Btree索引                                  |
| ---------------------------------------- | ---------------------------------------- |
| `=`, `<=>`, `IN()`, `IS NULL`, ` IS NOT NULL` | =, <=>, IN(), IS NULL, or IS NOT NULL,>, <, >=, <=, BETWEEN, !=, or <>,当通过like进行前缀匹配时，Btree的前缀索引也是有效的 |

通过上面的表格可以看出，明显Btree索引支持的筛选符号更多，所以Mysql默认Btree作为索引的存储结构

**注意上面Where符号生效的前提是限定条件为常量！**

+ 上述说明中的“ 常数值 ”表示以下内容之一：

 + 来自查询字符串的常量

 + 来自同一个连接的 一列const 或一列system

 + 一个不相关的子查询的结果

 + 任何完全由前面类型的子表达式组成的表达式


+ 范围条件提取算法可以处理任意深度的AND/ OR嵌套 ,其结果集输出不依赖于条件出现在WHERE子句中的顺序 。

要使优化器使用范围扫描，查询必须满足以下条件：

+ 只使用IN()，而不是NOT IN()。
+ 在IN()的左侧 ，行构造函数只包含列引用。
+ 在IN()的右侧 ，行构造函数只包含运行时常量，它们是在执行期间绑定到常量的文字或本地列引用。
+ 在IN()的右侧 ，有多个行构造函数。

**前段时间，我所在项目里面就遇到这样问题，当时的查询语句类似 ： select * from a where a.id in (select a.id from b where b.key = xxxx ) , 当时一直觉的没问题，毕竟子查询的结果作为结果集，但是explain查询的 type 是ALL，就是说进行了全表的扫描，很郁闷。最后改成了联查，就没了问题**

###  Index Merge Optimization

索引合并优化吗，索引合并只会在同一张表上生效，多表查询时，这样的优化不会生效

1. 如果您的查询具有WHERE 子语句中有比较深AND/ OR嵌套， 如果MySQL没有选择最佳的索引优化计划，可以用下面的方式简化查询条件


	(x AND y) OR z => (x OR z) AND (y OR z)
	
	(x OR y) AND z => (x AND z) OR (y AND z)


2. 索引合并不适用于全文索引。

索引合并有几种实现方式，在对select语句进行expalain时，索引使用情况可以可能会有下面几种可能的结果

+ Using intersect(...)

+ Using union(...)

+ Using sort_union(...)

##### intersection access algorithm

当多个where条件，通过and联合起来，并且每个condition 需要满足下面的条件之一

1. 所有的condition的键，都被index覆盖，也就是说，作为条件的列，必须要含有索引，并且都是单独的索引
2. 根据主键查询时，查询的表需要是InnoDB存储引擎 

当该WHERE子句转换为多个范围条件组合 时，该访问算法适用 OR，但索引合并联算法不适用。


**查询时使用索引，并不一定会完全避免全表扫描，避免全表扫描的**

### ICP （since version 5.6）

参考：

1. http://blog.codinglabs.org/articles/index-condition-pushdown.html
2. http://www.cnblogs.com/zhoujinyi/archive/2013/04/16/3016223.html 

ICP的原理简单说来就是将可以利用索引筛选的where条件在存储引擎一侧进行筛选，而不是将所有index access的结果取出放在server端进行where筛选。

但是需要注意的是：

1. 如果索引的第一个字段的查询就是没有边界的比如 key_part1 like '%xxx%'，那么不要说ICP，就连索引都会没法利用。
2. 如果select的字段全部在索引里面，那么就是直接的index scan了，没有必要什么ICP。

ICP的使用限制

1 当sql需要全表访问时,ICP的优化策略可用于range, ref, eq_ref,  ref_or_null 类型的访问数据方法 。
2 支持InnoDB和MyISAM表。
3 ICP只能用于二级索引，不能用于主索引。
4 并非全部where条件都可以用ICP筛选。
   如果where条件的字段不在索引列中,还是要读取整表的记录到server端做where过滤。
5 ICP的加速效果取决于在存储引擎内通过ICP筛选掉的数据的比例。
6 5.6 版本的不支持分表的ICP 功能，5.7 版本的开始支持。
7 当sql 使用覆盖索引时，不支持ICP 优化方法。

### 联查优化

对于 `select A left join B join_conditions where ....` 对于这样的语句注意下面几点

1. 左连接条件（on 后续的条件）来决定怎样从表B中retrieve数据，也就是说，就算Where Condition有对B表进行数据筛选的条件，也不会生效
2. 对于左查询，下面的 ` SELECT * FROM t1 LEFT JOIN t2 ON (column1) WHERE t2.column2=5`如果t2的列column1为null，那么where的条件就会没用，可以将上面的查询替换为`SELECT * FROM t1, t2 WHERE t2.column2=5 AND t1.column1=t2.column1`..通过这样的方式，mysql 优化器可以先筛选出t2满足 where 条件的行，然后再和t1通过条件进行联查 


### 多重范围读的优化（muti-range raed optimization）

+ 在一个二级索引上进行范围查找时会导致很多随机的磁盘读，利用MRR技术，MYSQL首先通过扫描索引，将相关联的行的主键取出。然后将键值进行排序，最后通过已经排序的主键获取到相对应的行。MRR的目的是通过

### [Order By 优化](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)

###### 通过索引进行Order by

利用索引进行排序的例子：

	SELECT * FROM t1
	  ORDER BY key_part1, key_part2;
	
	SELECT * FROM t1
	  WHERE key_part1 = constant
	  ORDER BY key_part2;
	
	SELECT * FROM t1
	  ORDER BY key_part1 DESC, key_part2 DESC;
	
	SELECT * FROM t1
	  WHERE key_part1 = 1
	  ORDER BY key_part1 DESC, key_part2 DESC;
	
	SELECT * FROM t1
	  WHERE key_part1 > constant
	  ORDER BY key_part1 ASC;
	
	SELECT * FROM t1
	  WHERE key_part1 < constant
	  ORDER BY key_part1 DESC;
	
	SELECT * FROM t1
	  WHERE key_part1 = constant1 AND key_part2 > constant2
	  ORDER BY key_part2;


**上面的`key_part1`,`key_part2`指的是联合索引的第一列部分，和第二列部分......**

注意，查询时使用索引并不意味着查询就通过Order by进行排序，同样的，就算查询时索引没用用上，排序时，索引也可能会用的上的

**通过索引进行排序，如果涉及多个列，可以将排序的列合并成一个键值**

**下面这些查询的排序都是没通过索引进行排序的**

1. 利用不同的索引<br/>  `SELECT * FROM t1 ORDER BY key1, key2;`

2. 利用联合索引中连续的部分作为排序条件 <br/>`SELECT * FROM t1 WHERE key2=constant ORDER BY key_part1, key_part3;`

3. 降序和升序混合着用<br/> `SELECT * FROM t1 ORDER BY key_part1 DESC, key_part2 ASC;`

4. where条件中的索引和order by中的索引不一致<br/> `SELECT * FROM t1 WHERE key2=constant ORDER BY key1;`


5. order by中含有不止索引的条件<br/>`SELECT * FROM t1 ORDER BY ABS(key);`  <br/>
  `SELECT * FROM t1 ORDER BY -key;`

6. 查询中有不同的order by 和 group by 语句

7. 查询时联合了多张表，但是 order by中的列并未全部用来获取第一个非常量表
  3

`SELECT ABS(a) AS a FROM t1 ORDER BY a` 这个查询中，a 所在的索引是无法在排序中使用 的但是通过下面的方式，order by可以使用上 列 a所在的索引 `SELECT ABS(a) AS b FROM t1 ORDER BY a`


对于filesort ，sort buffer 等于sort_buffer_size  系统变量指定的值。可以通过扩大指定，来增加排序的效率

**original filesort 算法**

这种排序算法弊端在于，这种排序是基于列的id的，也就是说，第一次通过where条件获取到值以后，然后通过 order by中的排序字段和行数据指针进行排序，然后第二次还要进行通过行数据的指针读取表获取到想要的数据

**这种算法的弊端在于会对表所在的磁盘进行两次随机的读，如果表很大，会导致读取效率低下**

**Modified filesort**

通过直接读取order by 和需要的row 值，使得他们形成一个tuples，而不是像上面`origin filesort`那样，将order by 列和row的指针合并成tuples。通过这样的方式，可以避免origin filesort的第二次的随机硬盘读取

注意： modified filesort的使用受限于[sysvar_max_length_for_sort_data](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_length_for_sort_data)这个系统变量。当这个变量设置过大，会导致磁盘的高频读写和cpu的运行缓慢。

**The In-Memory filesort Algorithm**

例如像这样的查询`SELECT ... FROM single_table ... ORDER BY non_index_column [DESC] LIMIT [M,]N` 由于查询结果只有几条数据，结果集所占的内存足以小于[sort_buffer_size](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sort_buffer_size)的值。这样的语句是可以直接在内存中进行排序的，而不用通过filesort     **其实也是上面filesort的一个特殊情况而已**

##### order by 优化的一些策略
1. 对于一些没有使用filesort的运行很慢的order by语句，可以降低阈值[sysvar_max_length_for_sort_data](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_length_for_sort_data)用来触发filesort
2. 为了加速order by的速度，看是否可以直接利用索引进行排序，如果不能，参考下面的策略
  1. 增加阈值[sysvar_sort_buffer_size](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sort_buffer_size)，可以使得结果集直接在内存中进行排序而不是用外排序，
  2. 减少每个column的内存占用，比如说：如果一个列的的长度永远不会超过16个字节，那么最好使用varchar(16)来存储，而不是varchar(200)
  3. 将[tempDir](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_tmpdir)指向的存储空间足够大的临时文件的路径，因为排序的时候会用到这些临时文件，文件足够大，可以减少

##### 查看order by的执行

通过执行查看`explain select ... order by`语句，可以判断是否语句在排序的时是否利用了索引，如果在explain的结果集里面的extra列看到 `using filesort`的话，可以判断这条语句没有通过索引来进行排序。
查看[Optimizing Queries with EXPLAIN](https://dev.mysql.com/doc/refman/5.7/en/using-explain.html)

在 Mysql 5.6.3版本以后，可以通过Optimizer-tracing来trace order by的filesort的信息


	Turn tracing on (it's off by default):
	SET optimizer_trace="enabled=on";
	SELECT ...; # your query here
	SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;
	# possibly more queries...
	# When done with tracing, disable it:
	SET optimizer_trace="enabled=off";
### [Group By 的优化](https://dev.mysql.com/doc/refman/5.7/en/group-by-optimization.html)

1. 首先，group by可以使用索引来加快执行速度，在mysql 中group by其实也是一种排序，所以在order by中优化算法适用于group by语句，
2. group by 使用排序来读取数据，所以只能用btree索引，不能使用在hash索引的算法中，
3. 如果利用 group by使用了到了索引，explain select 中的extra列是看不到using filesort的
4. **在你的查询中，对于没有排序要求的group by查询，[在group by后面加上order by null会提高性能](https://stackoverflow.com/questions/5231907/order-by-null-in-mysql)**
5. group by访问索引的方法：1 ： 松散索引扫描(Loose index scan)，2：紧索引扫描(tight index scan)

##### 松散索引扫描

 松散索引扫描的原理，直接访问相应的索引，不用排序就能根据索引来读取需要的数据，而对于如聚簇索引，我们可以读取前面的一部分的字段索引来获取数据，而不用满足所有的列，这就叫松散索引扫描

松散索引扫描的必要条件：
1. 查表只能对一个单表进行
2. group by使用索引为：对聚簇索引使用前缀索引
3. 使用类似group by 的操作的函数有distinct函数，使用此函数时，要么在一个索引上使用，要么在group by时，其group by的字句是索引扫描，否则会引起全表扫描。
4. 在使用group by语句中，如果使用聚合函数max(), min()等，如果列不在groupby的列中，或不在group by 列的聚簇索引的一部分，这将会用到排序操作
5. 只能对整个列的值排序时使用到索引，而只有前面一部分索引不能用到排序，如： 列 c1 char(20), index(c1(10))、这个只用了一半索引，将无法使用来对整个数据排序

### 避免查询时的全表扫描

利用explain对查询语句进行解剖后，会在结果集中，在type列里面的all类型都是经过全表扫描的，一般来说，出现全表扫描的原因有

1. 表很小，通过索引进行查询不如直接进行全表扫描来的快
2. 在on条件或者where条件下没有可用的索引列
3. 在查询时，通过索引进行比较后，会出现覆盖大量数据集的情况，这个时候不如直接进行全表扫描来的快比如说
4. 使用一个低分辨率的索引，然后用这个索引去匹配查询条件时，到时可匹配的列过多，不如直接扫描全表来的快

 **使用ANALYZE TABLE语法对表进行分析，利用force index句式强制使用index进行扫描**

	SELECT * FROM t1, t2 FORCE INDEX (index_for_column) WHERE t1.col_name=t2.col_name;


Start mysqld with the --max-seeks-for-key=1000 option or use SET max_seeks_for_key=1000 to tell the optimizer to assume that no key scan causes more than 1,000 key seeks. See Section 5.1.5, “Server System Variables”.




