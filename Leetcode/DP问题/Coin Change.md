#  Coin Change 

## 问题描述

You are given coins of different denominations and a total amount of money *amount*. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return `-1`.

**Example 1:**

```
Input: coins = [1, 2, 5], amount = 11
Output: 3 
Explanation: 11 = 5 + 5 + 1
```

**Example 2:**

```
Input: coins = [2], amount = 3
Output: -1
```

**Note**:
You may assume that you have an infinite number of each kind of coin.

## 解题思路

这是一个找零问题，也是完全背包问题的一个变型,回顾一下完全背包问题的状态转移公式和伪代码

$F[i][v] = Max(F[i-1][v],F[i-1][v - cost[i]] + weight[i])$ 

```
for i...N
  for 0...V
   f[v] = max(f[v],f[v-c[i]] + w[i])
```

在这个问题里面，可以将看作为coin作为无限的物品，将最少的coin作为价值。

$F[i][j] = min(F[i-1][j - coin[i]] + 1 , F[i-1][j])$，这个即为其状态转移方程，因为我们要取最小值，所以边界和初始化值应该设为Max

初始化值

```

```



伪代码：

```

for c in coins:
	f[c] = 1;
for coin in coins：
  for a < amount:
    F[a] = min(F[a-coin] + 1,F[a-coin])
```

例如，如果$F[a1]$ 恰好能再$a1 - coin$ 的基础上找零，那么$F[a1]$ 就是$F[a1 - coin] + 1$ ， 因为我们有初始化值，依次类推即可。 

```go
func coinChange(coins []int, amount int) int {
    if amount == 0{
		return 0
	}
	
	dp := make([]int,amount+1)
	for index,_ := range dp{
		dp[index] = math.MaxInt32;
	}
	for _,value := range  coins {
		if value < len(dp){
			dp[value] = 1
		}
	}
    
	for _,value := range  coins {
	    for index := 2; index <= amount; index++{	
			if index >= value{
				dp[index] = Min(dp[index - value] + 1,dp[index])
			}
		}
	}
	if (dp[amount] == math.MaxInt32){
		return -1
	}else {
		return dp[amount]
	}
}
func Min(a int ,b int) int {
	if a < b {
		return a
	}else {
		return b
	}
}
```

