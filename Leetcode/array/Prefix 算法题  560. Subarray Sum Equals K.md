# 560. Subarray Sum Equals K

Given an array of integers and an integer **k**, you need to find the total number of continuous subarrays whose sum equals to **k**.

**Example 1:**

```
Input:nums = [1,1,1], k = 2
Output: 2
```





这也是个经典的题了



使用 prefix 算法

所谓 prefix 算法 即为 将前 n 个数向加得到的结果记录起来（map 或者 array）

对于这道题来说有

给定一个 num = [1,2,3,2,5,7,7], k =  7 其结果显然是 n := 4

1. 计算其 prefix sum  有 [1,3,6,8,15,22]
2. 将 prefix sum 用 map映射起来，value 为 prefix sum 值，value 是出现的次数
3. 将prefix sum 重新遍历一遍即可得到结果

```go
func subarraySum(nums []int, k int) int {
    preSumMap := make(map[int]int)
    result := 0
    prefixSum := 0
    preSumMap[0] = 1
    for _,value := range nums{
        prefixSum += value
        result += preSumMap[prefixSum - k]
        preSumMap[prefixSum]++
    }
    return result
}
```



**下面是我的思考的结果（错误）**

```go
func subarraySum(nums []int, k int) int {
    preSumMap := make(map[int]bool)
    result := 0
    prefixSum := 0
    preSumMap[0] = true
    for _,value := range nums{
        prefixSum += value
        if preSumMap[prefixSum - k] {
            result++
        }
        preSumMap[prefixSum] = true
    }
    return result
}
```

这个 通不过 测试案例

[0,0,0,0,0,0,0,0,0,0]
0

result = 10

expect = 55

原因在于没有记录 prefix sum 出现的次数，



