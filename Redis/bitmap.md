# Bitmap

## BITMAP底层数据结构

bitmap其实就是一个字符串，它可以对字符串进行位操作。

## 命令示例

除了 SETBIT 和GETBIT ，BITMAP的所有命令的 时间复杂度都是`O(N)`。

```bash
10.199.5.217:6379> set test aaaa
OK
# 获取第10个bit的值
10.199.5.217:6379> getbit test 10
(integer) 1
## 越界，获取到的是0
10.199.5.217:6379> getbit test 1000000
(integer) 0
# 将第8位bit 设置成0
10.199.5.217:6379> setbit test 8 1
(integer) 0
# 再获取值，已经变化了
10.199.5.217:6379> get test
"a\xe1aa"
# test 中 bit位为1的个数
10.199.5.217:6379> bitcount test
(integer) 13
# 第一个 bit位为0的位置
10.199.5.217:6379> bitpos test 0
(integer) 0
# 第一个 bit位为1的位置
10.199.5.217:6379> bitpos test 1
(integer) 1
```

## 适用场景

### 已登录用户统计

利用UserId 作为 bit的位偏移量，登录时设置为1，登出设置为0，加上超时时间。但是这个前提是UserID是范围可控的数字。