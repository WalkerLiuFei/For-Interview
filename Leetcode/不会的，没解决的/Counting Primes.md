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

3. 对于以上的埃氏素数标记，如果，考虑下面的情况

    1. 3 * 5 = 15
    2. 5 * 3 = 15 

    可以看到会出现重复筛选，怎么解决这种问题?可以通过，从 $p^2$ 开始，步长为p这样进行标记。

4. 避免循环的 开平方操作，使用 `j * j < i` 这种 比较方式，而不是`j < sqrt(i)`这种方式

5. 

答案代码：

```java
public int countPrimes(int n) {
   boolean[] isPrime = new boolean[n];
   for (int i = 2; i < n; i++) {
      isPrime[i] = true;
   }
   // Loop's ending condition is i * i < n instead of i < sqrt(n)
   // to avoid repeatedly calling an expensive function sqrt().
   for (int i = 2; i * i < n; i++) {
      if (!isPrime[i]) continue;
      for (int j = i * i; j < n; j += i) {
         isPrime[j] = false;
      }
   }
   int count = 0;
   for (int i = 2; i < n; i++) {
      if (isPrime[i]) count++;
   }
   return count;
}
```

