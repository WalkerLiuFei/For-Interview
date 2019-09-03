# Hash表操作命令



## HSET

> 时间复杂度： O(1)

将哈希表 `hash` 中域 `field` 的值设置为 `value` 。

如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 `HSET` 操作。

如果域 `field` 已经存在于哈希表中， 那么它的旧值将被新值 `value` 覆盖。

```bash
redis> HSET website google "www.g.cn"
(integer) 1

redis> HGET website google
"www.g.cn"

# 更新

redis> HSET website google "www.google.com"
(integer) 0

redis> HGET website google
"www.google.com"
```



## HSETNX

> 时间复杂度： O(1)

当且仅当域 `field` 尚未存在于哈希表的情况下， 将它的值设置为 `value` 。

如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。

如果哈希表 `hash` 不存在， 那么一个新的哈希表将被创建并执行 `HSETNX` 命令。

## HDEL



## HEXISTS



## HLEN

返回哈希表 `key` 中域的数量。

## HSTRLEN

```bash
redis> HMSET myhash f1 "HelloWorld" f2 "99" f3 "-256"
OK

redis> HSTRLEN myhash f1
(integer) 10

redis> HSTRLEN myhash f2
(integer) 2

redis> HSTRLEN myhash f3
(integer) 4
```

## HINCRBY

为哈希表 `key` 中的域 `field` 的值加上增量 `increment` 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 `key` 不存在，一个新的哈希表被创建并执行 [HINCRBY](http://redisdoc.com/hash/hincrby.html#hincrby) 命令。



## HINCRBYFLOAT

。。。

## HMSET 

**HMSET key field value [field value …]**

同时将多个 `field-value` (域-值)对设置到哈希表 `key` 中。

此命令会覆盖哈希表中已存在的域。

```bash
redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HGET website google
"www.google.com"

redis> HGET website yahoo
"www.yahoo.com"
```

