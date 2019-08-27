# InnoDB数据结构

 在整理InnoDB存储引擎的索引的时候，发现B+树是离不开页面page的。所以先整理InnoDB的数据存储结构。

关键词：**Pages, Extents, Segments, and Tablespaces**

## 如何存储表

### 如何存储表**

MySQL 使用 InnoDB 存储表时，会将表的定义和 数据索引等信息分开存储，其中**表的定义存储在 .frm 文件中，数据索引存储在 .ibd 文件中**

**.frm **  ：  **.frm**来存储表的定义，包括字段定义，索引等。.frm 与具体的存储引擎无关

**.ibd** : InnoDB 中用于存储数据的文件总共有两个部分，一是系统表空间文件，包括 ibdata1、 ibdata2 等文件，其中存储了 InnoDB 系统信息和用户数据库表数据和索引，是所有表公用的。
当打开 innodb_file_per_table 选项时， **.ibd 文件就是每一个表独有的表空间，文件存储了当前表的数据和相关的索引数据**。



## .idb 文件的格式

.idb 在此文件中，存储与该表相关的数据、索引、表的内部数据字典信息。



![img](https://upload-images.jianshu.io/upload_images/3892388-10ffd7b8a9e6e395.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/711/format/webp)