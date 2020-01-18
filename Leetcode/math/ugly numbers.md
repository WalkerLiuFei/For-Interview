# Ugly Number

## 问题描述

Write a program to check whether a given number is an ugly number.

Ugly numbers are **positive numbers** whose prime factors only include `2, 3, 5`.

**Example 1:**

```
Input: 6
Output: true
Explanation: 6 = 2 × 3
```

**Example 2:**

```
Input: 8
Output: true
Explanation: 8 = 2 × 2 × 2
```

**Example 3:**

```
Input: 14
Output: false 
Explanation: 14 is not ugly since it includes another prime factor 7.
```

**Note:**

1. `1` is typically treated as an ugly number.
2. Input is within the 32-bit signed integer range: [−231, 231 − 1].

## 解题思路

这个题，是看了[count prime]( https://leetcode.com/problems/count-primes/ ) 才有的思路，通过快速的`埃氏筛选`进行

1. 首先需要一个循环，判断这个数是否可以被2，3，5整除，如果可以，则标记为true。**首先这一步，所有的素数都不会被标记到。**其他的还有这些素数的乘积等，例如 7 * 7 = 49，7 * 11 = 77等等

2.  然后通过类似埃氏筛选的方式判断这个数是否可以被其他数整除.注意，这个循环和判断素数

   ```
   for i < n:
     if i % 2 == 0 || i % 3 == 0 || i % 5 == 0:
       isUgly[i] = true
   for i = 7; i < n;i++
     if isUgly[i]
        continue
     for j = i; j  < n; j += i:
       isUgly[j] = false
       
   ```

3. 解释，当 n = 100 时

   1. i = 7 时，   14,21,28...63 ，这些为7倍数的都会被标记为false
   2. 当 i = 8 时，因为8是ugly number，所以跳过它，i = 9时一样 。。。。。
   3. 当 i = 11时，22，33，44...99.这些数会被标记为false

4.  对 步奏2 像 [count prime]( https://leetcode.com/problems/count-primes/ ) 去优化，例如:  7 * 11 和 11 * 7 这样会被重复计算，基于快速质数筛选，可以对外循环进行优化

   ```
   for i < n:
     if i % 2 == 0 || i % 3 == 0 || i % 5 == 0:
       isUgly[i] = true
   for i = 7; i * i< n;i++
     if isUgly[i]
        continue
     for j = i; j  < n; j += i:
       isUgly[j] = false
       
   ```







# Ugly 2

```go
func isUgly(num int) bool {
    isUgly := make([]bool,num)
    for i = 2;i < num;i++{
        if i % 2 == 0 || i % 3 == 0 || i % 5 == 0{
            isUgly[i] = true
        } 
    }
    for i := 0;i * i < n; i++{
        if isUgly[i]{
            continue
        }
        for j := i; j < n; j += i{
            ifUgly[j] = false
        }
    }
    count := 0
    for _,v := range isUgly{
        if v {
            count++
        }
    }
	return 
}
```

