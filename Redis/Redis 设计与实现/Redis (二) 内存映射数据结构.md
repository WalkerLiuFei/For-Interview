

# Redis 内存映射数据结构

## 整数集合

整数集合（intset）用于有序、无重复地保存多个整数值， 根据元素的值， 自动选择该用什么长度的整数类型来保存元素，一般使用最长元素的字长来进行保存。

### 数据结构

```c
typedef struct intset {

    // 保存元素所使用的类型的长度
    // 表示是 16，32，64 位 的整数
    // 根据添加的元素，自动升级编码
    uint32_t encoding;

    // 元素个数
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```

### 操作及特性

创建 `intset` 之后，就可以对它添加新元素了。

添加新元素到 `intset` 的工作由 `intset.c/intsetAdd` 函数完成，需要处理以下三种情况：

1. 元素已存在于集合，不做动作；
2. 元素不存在于集合，并且添加新元素并不需要升级；
3. 元素不存在于集合，但是要在升级之后，才能添加新元素；

特性

1.  intsetAdd在添加元素要保证 contents 中没有重复元素
2.   确保contents 数据有序
3. 不支持降级操作，也就是说 64 字长的int set 不支持向 16字长的int set 降级。

![操作流程](https://redisbook.readthedocs.io/en/latest/_images/graphviz-1565e65522fdcd4245030f17b5074729033297d8.svg)

### 小结

- Intset 用于有序、无重复地保存多个整数值，会根据元素的值，自动选择该用什么长度的整数类型来保存元素。
- 当一个位长度更长的整数值添加到 intset 时，需要对 intset 进行升级，新 intset 中每个元素的位长度，会等于新添加值的位长度，但原有元素的值不变。
- 升级会引起整个 intset 进行内存重分配，并移动集合中的所有元素，这个操作的复杂度为 O(N)O(N) 。
- Intset 只支持升级，不支持降级。
- Intset 是有序的，程序使用二分查找算法来实现查找操作，复杂度为 O(lgN)O(lg⁡N) 。

##  压缩列表

Ziplist 是为了尽可能地节约内存而设计的特殊编码双端链表，可以存储字符串值和整数值

### 数据结构

```
area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
                                       ^                          ^        ^
address                                |                          |        |
                                ZIPLIST_ENTRY_HEAD                |   ZIPLIST_ENTRY_END
                                                                  |
                                                         ZIPLIST_ENTRY_TAIL
```

| `zlbytes` | `uint32_t` | 整个 ziplist 占用的内存字节数，对 ziplist 进行内存重分配，或者计算末端时使用。 |
| --------- | ---------- | ------------------------------------------------------------ |
| `zltail`  | `uint32_t` | 到达 ziplist 表尾节点的偏移量。 通过这个偏移量，可以在不遍历整个 ziplist 的前提下，弹出表尾节点。 |
| `zllen`   | `uint16_t` | ziplist 中节点的数量。 当这个值小于 `UINT16_MAX` （`65535`）时，这个值就是 ziplist 中节点的数量； 当这个值等于 `UINT16_MAX` 时，节点的数量需要遍历整个 ziplist 才能计算得出。 |
| `entryX`  | `?`        | ziplist 所保存的节点，各个节点的长度根据内容而定。           |
| `zlend`   | `uint8_t`  | `255` 的二进制值 `1111 1111` （`UINT8_MAX`） ，用于标记 ziplist 的末端的偏移量 |

**因为 ziplist 由连续的内存块构成， 在最坏情况下， 当 `ziplistPush` 、 `ziplistDelete` 这类对节点进行增加或删除的函数之后， 程序需要执行一种称为连锁更新的动作来维持 ziplist 结构本身的性质， 所以这些函数的最坏复杂度都为 $O(N^2)$ ** 

#### entryX的结构

一个 ziplist 可以包含多个节点，每个节点可以划分为以下几个部分：

```
area        |<------------------- entry -------------------->|

            +------------------+----------+--------+---------+
component   | pre_entry_length | encoding | length | content |
            +------------------+----------+--------+---------+
```

1. `pre_entry_length` 记录了前一个节点的长度，通过这个值，可以进行指针计算，从而跳转到上一个节点。

2. `encoding ` 代表着编码方式

3. 






​     