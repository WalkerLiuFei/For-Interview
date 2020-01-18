# Counting Primes

  https://leetcode.com/problems/count-primes/ 

## 问题描述  

Count the number of prime numbers less than a non-negative number, ***n\***.

**Example:**

```
Input: 10
Output: 4
Explanation: There are 4 prime numbers less than 10, they are 2, 3, 5, 7.
```



## 解题思路（别人的）

1. 首先了解快速质数判断 ： 即判断n是否时素数，只需要判断

2. 埃氏素数筛选法 ： 见下

    ![img](https://leetcode.com/static/images/solutions/Sieve_of_Eratosthenes_animation.gif)

3. 避免循环的 开平方操作，使用 `j * j < i` 这种 比较方式，而不是`j < sqrt(i)`这种方式

伪代码：

```
for i < n:
   if not isPrime[i]:
      continue
   for j = 2; j * j < i;j ++:
      isPrime[i] = j % i != 0
      if not isPrime[i]:
         break
for v in isPrime:
   count++
count
         
```

