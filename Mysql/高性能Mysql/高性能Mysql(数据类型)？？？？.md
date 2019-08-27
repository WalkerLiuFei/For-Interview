## MYSQL 数据类型

1. **对于有些语句最后加‘\g’ 的含义是什么？**
2. **为什么列值最好不要为 null，如果为 null的话，对于该列对应的索引有何影响？**
3. **timestamp数据类型相对于datetime有何优势，有何劣势？**
4. **怎样使用吗，枚举类型代替字符串类型，那些应用场景更合适？**
5. **对于MYSQL 来说 int(1) 和int(20)有什么区别吗？**
6. **DECIMAL数据类型的作用**
7. **varchar和char型的区别，各自适用的业务场景**
8. **BOLB和TEXT的存储方式？ 排序的时候，这两种数据类型的特殊之处**
9. **标识列的数据类型选择**
10. **Mysql表的列太多会导致的问题**
11. **查询中太多的关联语句会造成的性能问题**
12. **第一范式，第二范式，第三范式，怎样分析业务表中存在不适于范式的设计？范式化和反范式化各自的优势**
13. **缓存表和汇总表带来的性能提升？**
14. **什么是计数器表？业务场景是什么?**



## 答案

1. 省略
2. 列值最好不要为Null的原因包括 ： 
   1.  空值是不占空间的，而`null`值 是占空间的 （一个 bit的空间）！
   2.  B-tree 索引不会存储NULL 值，所以如果索引的字段可以为 null的话，索引的效率就会降低
   3.  如果有 Null column 存在的情况下，count(Null column)需要格外注意，``null` `值不会参与统计 
   4.  对于null 列的筛选， `<> ` 满足不了所有的筛选条件
3. Timestamp 只需要4个字节，Datetime需要8个字节。 但是后者可以表示的时间范围更广。Timestamp 不用担心时区问题。显示问题，datetime 的前端显示更为友好
4. 省略
5. The `x` in `INT(x)` has nothing to do with space requirements or any other performance issues, it's really just the *display width*. Generally setting the display widths to a reasonable value is mostly useful with the `UNSIGNED ZEROFILL` option.所以在建表时 指定INT(11) 中操作是没有意义的。
6. Decimal 实际是以字符串的形式存放的，Decimal 可以指定精度。
7. varchar和 char 类型的可变长度字符串，其需要多余的字节来记录长度。另外，**varchar会保留末尾空格，而char则不会**
8. Mysql将 BOLB 和 TEXT作为一个单独的对象进行处理，当他们的太大时， 都是通过以标识额外存储文件路的方式来记录数据的
   1. BLOB 和 TEXT 列都不能有默认值
   2. 对于BLOB和TEXT列的索引，必须指定索引前缀的长度
9. 最好使用整型来作为标识列，因为在上面可以很方便的建立索引，并且可以使用AUTO_INCREMENT。\
10. 范式化的带来的优势 ：
   1. 表的业务属性更清晰
   2. 范式化的更新操作通常比反范式的块
   3. 当数据较好的范式化，只有少数的重复数据，所以只需要修改更少的数据
   4. 范式化的表通常更新，更容易放到内存中，所以执行操作比较快
   5. 很少有多余的数据意味着检索列表数据时更少需要Distinct 或者 Group by语句。
11. 反范式化带来的优势
    1. 很好的避免关联查询
    2. 业务实现更简单
12. 最好的方式是混用范式化和反范式。
13. 优化 `ALTER TABLE ` 操作，在进行`ALTER TABLE `操作时，一般会造成表的不可用，
    1. 主备切换 ： 在备用表上面进行表结构更新，然后将备用表切换成master
    2. 影子拷贝 ： 用要求的表结构创建一张新表，然后通过重命名和删表的方式进行替换。
14. 