---
title: Mysql的性能优化（存储篇）
---

# Mysql 存储引擎概述
Mysql 5.0支持的引擎包括MyISAM,InnoDB,BDB等，其中InnoDB和BDB提供事务安全表。其他引擎都是提供的非事务安全表。MySql 5.5以后的默认引擎是InnoDB.

在创建新表时可以通过ENGINE关键字来指定存储引擎。

```
CREATE TABLE COUNTRY(name=varchar(50) primary key) ENGINE = INNODB
```

可以通过查看表的创建信息来查看表的引擎（其中一种方式）

```
show create table 表名
```

## 选择合适的存储引擎

### MyISAM
 不支持事务，不支持外键，优势是访问速度快，对事务完整性没有要求或者以SELECT,INSERT为主的应用选择MyISAM引擎较为合适

存储的组成部分，在创建表时，可以通过DATA DIRECTORY 和 INDEX DIRECTORY指定存储路径的绝对路径

+ .frm(存储表定义)
+ .MYD(存储数据
+ .MYI(存储索引)

MyISAM的表的3中存储格式

+ 静态（固定长度）表
+ 动态表
+ 压缩表

静态表时默认的存储格式。优点是存储迅速，容易缓存，出现故障容易恢复。缺点是字段的字长都是固定的，占用空间比较大，另外，在存储的时候会自动去掉尾部的空格。

动态表包含变长的字段，记录的数据固定，这样存储的优点是占用的空间相对较小。但是频繁的更新和删除数据会产生碎片，需要定期执行OPTIMIZE TABLE 语句。或者myismchk -r命令来改善性能，并且在出现故障时难以恢复。

### InnoDB

InnoDB具有提交，回滚和崩溃恢复能力的事务安全。InnoDB处理效率差，并且占用更多的磁盘空间以保存数据和索引。

特点

+ InnoDB的自动增长列（auto_increment）插入0或者null的结果其实是插入的将是自动增长后的值。
+ 外检约束： 在创建索引时，可以指定在删除，更新父表时，对字表进行的相应操作。
 + RESTRICT，NO Action : 限制在字表有关联的情况下父表不能更新
 + CASCADE：表示父表在更新数据或删除数据时，更新字表或删除字表的对应记录。
 + SET NULL：表示父表在更新数据或删除数据时，字表致null

存储：
+ 使用共享空间： 表结构保存在.frm文件中，数据和索引保存在innodb_data_home_dir和innodb_data_file_path定义的表空间中
+ 使用多表空间： 表结构保存在.frm文件中，每个表的数据和索引单独保存在.idb中。如果是个分区表，则每个分区对应单独的.idb文件，文件名是“表名+分区名”，可以在创建分区的时候指定每个分区的数据文件的位置，以此来将表的IO均匀分布在多个磁盘中。

```Sql

CREATE TABLE COUNTRY(
	country_id smallint unsigned not null auto_increment,
    country varchar(50) NOT NUll,
    last_update timestamp not null default current_timestamp on update current_timestamp,
    primary key(country_id)
)engine=InnoDB default charset=utf8;

create table city (
	city_id smallint unsigned not null auto_increment,
    city varchar(50) not null,
    country_id smallint unsigned not null,
    last_update timestamp not null default current_timestamp on update current_timestamp,
    primary key(city_id),
    foreign key (country_id) references country (country_id)     
    on delete restrict  on update cascade  //主表在删除记录时限制，在update时不限制....
)engine = InnoDB default charset=utf8;

```   

### 如何选择合适的存储引擎

综上所述，

+ MyISAM: 应用以读和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性，并发性要求不高。
+ InnoDB: 用于事务处理的应用程序，支持外键。如果应用对事务的完整性要求比较高。并且有大量的插入和更新操作，并且对于事务的的完整提交和回滚有要求。
 
# 选择合适的数据类型

## varchar 和char

varchar 和char 名字看起来varchar多了个前缀var，也就是说varchar的长度是可变的，其占用的存储空间可以是动态的，相较于char固定长度的数据类型，占用的空间更小。

+ MyIASM引擎的表，建议使用char类型数据，对于InnoDB,建议使用varchar类型的数据。

## BLOB和TEXT

主要区别在于BLOB可以存储二进制数据，例如图片。而TEXT只能存储字符数据。对于存有BLOB或者TEXT的表，在大量的删除和更新操作后，会造成存储空间上的空洞，**要经常使用OPTIMIZE TABLE来对表进行碎片整理**。

   -**以上只针对MyISAM 引擎的数据表，对于InnoDB的数据表支持optimize 命令,但是有问题，参考——>。<a href = "http://stackoverflow.com/questions/30635603/what-does-table-does-not-support-optimize-doing-recreate-analyze-instead-me">stackoverflow</a>**-

使用合成的Synthetic 索引来提高大文本字段的的查询性能。**利用大文本数据形成一个Hash值保存在一个单独的字段中，通过检索字段来检索文本。**  

避免检索大量的文本内容，将Blob和TEXT文本保存在单独的表中或者NOSQL数据库中。

## 总结

+ 对于字符类型，要根据存储引擎来进行相应的选择。
+ 对精度要求较高的应用中，建议使用定点数来存储数值，以保证结果的准确性
+ 对含有TEXT和BLOB字段的表，如果经常做删除和修改记录的操作要定时执行**OPTIMIZE TABLE**功能对表进行碎片整理
+ 日期类型要根据实际需要选择满足应用最小的存储类型
