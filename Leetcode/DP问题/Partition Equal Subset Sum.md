#   Partition Equal Subset Sum 

# 问题描述

Given a **non-empty** array containing **only positive integers**, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.

**Note:**

1. Each of the array element will not exceed 100.
2. The array size will not exceed 200.

 

**Example 1:**

```
Input: [1, 5, 11, 5]

Output: true

Explanation: The array can be partitioned as [1, 5, 5] and [11].
```

 

**Example 2:**

```
Input: [1, 2, 3, 5]

Output: false

Explanation: The array cannot be partitioned into equal sum subsets.
```

## 自己的思路（部分）

其实这个思路应该是更好理解的，设一个数组 

```
dp = int[sum]
dp[0] = true
for num := num[i]
 for i = 0; i < sum; i++{
   if dp[i] {
    	dp[i+sum] = true
   }
 } 
```

所以这个解法的时间复杂度是`O(N * sum)` ，



## 解题思路（别人的）

DP问题，可以通过0/1背包问题来解决。回顾一下`01`背包问题的状态转移方程：V是体积，w[i]是价值，费用是c[i]

$F[i][v] = Max(F[i-1][v],F[i-1][v-c[i] + w[i])$  

现在可以这样考虑，`01`背包问题和这个问题的相似性，这个问题没有“ 价值 ”，只总的重量.

可以定义一个二维变量将,$F[i][j]$ , 其中`j` 表示 从 `0 - i`组成的和，  

$F[i][j] = F[i - 1][j - nums[i]]$  表示 not take nums中第i个元素， $F[i][j] = F[i - 1][j]$ 表示 take第i个元素，令

$F[i][j] = F[i-1][j-nums[i]] \ ||  \ F[i-1][j]$  即为需要的状态转移方程，根据`01`背包问题，可以将其简化为1维数组

$F[j] = F[j - nums[i]] || F[j]$   最后的这个状态转移方程即为最后的解 