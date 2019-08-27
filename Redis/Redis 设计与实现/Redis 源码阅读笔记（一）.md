Redis内部数据结构

## 字符串

Redis通过 动态字符串来实现的 字符串功能

### SDS 数据结构

```c
struct sdshdr {

    // buf 已占用长度
    int len;

    // buf 剩余可用长度
    int free;

    // 实际保存字符串数据的地方
    char buf[];
};
```



### SDS 的特性

1. 二进制安全 ： C字符串的数据肯定满足某种编码的，而SDS不需要。
2. 获取字符串长度 的时间复杂度为 O(1)，因为有一个专门的记录长度的属性，如果使用使用原始的`C`字符串，计算长度的复杂度为`O(N)`。
3. 避免缓存区溢出 ： 比如两个字符串相 concat，C字符串如果目标字符串没有提前分配足够的空间，这个操作就会造成缓冲区溢出，而SDS的API封装了这些检查和分配策略。
4. 内存开销小，对于经常变化的字符串，有预分配的策略
   1. < 1MB ，预分配内存大小为需要内存的两倍
   2. /> 1MB , 多分配1MB空间
   3. 预分配的内存，在字符串被删除之前，不会被删除
   4. **这里需要注意的点是，对于经常append 的字符串，需要及时释放多余的预分配内存**



## 双端链表

Redis通过 双端链表和压缩列表实现的集合功能

### 数据结构

```c
typedef struct list {

    // 表头指针
    listNode *head;

    // 表尾指针
    listNode *tail;

    // 节点数量
    unsigned long len;

    // 复制函数
    void *(*dup)(void *ptr);
    // 释放函数
    void (*free)(void *ptr);
    // 比对函数
    int (*match)(void *ptr, void *key);
} list;
```

```c
typedef struct listNode {

    // 前驱节点
    struct listNode *prev;

    // 后继节点
    struct listNode *next;

    // 值
    void *value;

} listNode;
```





Redis 有两种实现 **双端链表**，**压缩列表**。 **Redis在创建集合时默认使用压缩列表的数据结构，只有在不得不适用双端链表时在使用双端链表。**

- 事务模块使用双端链表依序保存输入的命令；
- 服务器模块使用双端链表来保存多个客户端；
- 订阅/发送模块使用双端链表来保存订阅模式的多个客户端；
- 事件模块使用双端链表来保存时间事件（time event）；

### 相关的操作

列表的先关操作底层都是通过链表实现的。

XPUSH ， XPOP，等等



## 字典

参考 ： https://redisbook.readthedocs.io/en/latest/internal-datastruct/dict.html

### 数据结构

Redis 的 Hash 类型键使用以下两种数据结构作为底层实现:

1. 字典；
2. [压缩列表](https://redisbook.readthedocs.io/en/latest/compress-datastruct/ziplist.html#ziplist-chapter)；

字典的定义 ： 

```c
/*
 * 字典
 *
 * 每个字典使用两个哈希表，用于实现渐进式 rehash
 */
typedef struct dict {

    // 特定于类型的处理函数
    dictType *type;

    // 类型处理函数的私有数据
    void *privdata;

    // 哈希表（2 个）
    dictht ht[2];

    // 记录 rehash 进度的标志，值为 -1 表示 rehash 未进行
    int rehashidx;

    // 当前正在运作的安全迭代器数量
    int iterators;

} dict;
```



Hash 表的数据结构 ：

```c
/*
 * 哈希表
 */
typedef struct dictht {

    // 哈希表节点指针数组（俗称桶，bucket）
    dictEntry **table;

    // 指针数组的大小
    unsigned long size;

    // 指针数组的长度掩码，用于计算索引值
    unsigned long sizemask;

    // 哈希表现有的节点数量
    unsigned long used;

} dictht;
```

Hash 表节点的数据结构

```c
/*
 * 哈希表节点
 */
typedef struct dictEntry {

    // 键
    void *key;

    // 值
    union {
        // 键值可以是一个指针，*val标识
        void *val;
        // 或是一个uint64_t 的整数
        uint64_t u64;
        // 或是一个int64_t 的整数
        int64_t s64;
    } v;

    // 链往后继节点
    struct dictEntry *next;

} dictEntry;
```

### 细节

1. 通过实现可以了解到，`dictht` [使用链地址法来处理键碰撞](http://en.wikipedia.org/wiki/Hash_table#Separate_chaining)： 当多个不同的键拥有相同的哈希值时，哈希表用一个链表将这些键连接起来。**同Java的实现一致** ![Redis 字典结构](https://redisbook.readthedocs.io/en/latest/_images/graphviz-6989792733a041b23cdc0b8f126434590c50a4e4.svg)
2. Redis 目前使用两种不同的哈希算法，对于使用何种算法，取决于具体处理的数据
   1. MurmurHash2 32 bit 算法
   2. 基于 djb 算法实现的一个大小写无关散列算法：具体信息请参考 <http://www.cse.yorku.ca/~oz/hash.html> 。
3. 字典的数据结构有两张Hash表，对于分配空间的问题
   1. `ht[0]->table` 的空间分配将在第一次往字典添加键值对时进行；
   2. `ht[1]->table` 的空间分配将在 rehash 开始时进行；

### Rehash 

Rehash 的过程 ： 

1. 创建一个比 `ht[0]->table` 更大的 `ht[1]->table` ；
2. 将 `ht[0]->table` 中的所有键值对迁移到 `ht[1]->table` ；
3. 将原有 `ht[0]` 的数据清空，并将 `ht[1]` 替换为新的 `ht[0]` ；
4. 新建一个新的Hash表，替换原来的ht[1]

Rehash 的特点

1. Rehash 是分多次，渐进式的，因为Rehash时会将hashtable加锁



Rehash的实质是对HashMap 进行扩容

1. 触发条件 ：

   1. used / size  >= 1  && dict_can_resize 为真时
   2. used / size  >= 5 时， 这种情况一般是 Hash表中出现了较多的Hash 冲突，造成了Hash表的访问效率降低（同一个槽出现了链表形式的数据结构）

2. 什么时候dict_can_resize 为假 ？ 

   **在前面介绍字典的应用时也说到过， 数据库就是字典， 数据库里的哈希类型键也是字典， 当 Redis 使用子进程对数据库执行后台持久化任务时（比如执行 `BGSAVE` 或 `BGREWRITEAOF` 时）， 为了最大化地利用系统的 [copy on write](http://en.wikipedia.org/wiki/Copy-on-write) 机制， 程序会暂时将 `dict_can_resize` 设为假， 避免执行自然 rehash ， 从而减少程序对内存的触碰（touch）。当持久化任务完成之后， `dict_can_resize` 会重新被设为真。另一方面， 当字典满足了强制 rehash 的条件时， 即使 `dict_can_resize` 不为真（有 `BGSAVE` 或 `BGREWRITEAOF` 正在执行）， 这个字典一样会被 rehash**

3. 在refresh 期间，

### 插入流程图



![Redis 字典插入的流程](https://redisbook.readthedocs.io/en/latest/_images/graphviz-68f4129c529e0c49d38cfe664cad48af4412770a.svg)



## 跳跃表

跳跃表在Redis中 用作有序列表的实现。

关于跳跃表这种数据结构 ：[参考](https://lotabout.me/2018/skip-list/) 

### 数据结构

```c
typedef struct zskiplist {

    // 头节点，尾节点
    struct zskiplistNode *header, *tail;

    // 节点数量
    unsigned long length;

    // 目前表内节点的最大层数
    int level;

} zskiplist;

```



```c
typedef struct zskiplistNode {

    // member 对象
    robj *obj;

    // 分值
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 这个层跨越的节点数量
        unsigned int span;

    } level[];

} zskiplistNode;
```

