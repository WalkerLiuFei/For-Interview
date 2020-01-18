#   Best Time to Buy and Sell Stock with Cooldown 

## 问题描述

Say you have an array for which the *i* th element is the price of a given stock on day *i*.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:

- You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
- After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day)

**Example:**

```
Input: [1,2,3,0,2]
Output: 3 
Explanation: transactions = [buy, sell, cooldown, buy, sell]
```

Say you have an array for which the *i*th element is the price of a given stock on day *i*.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:

- You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
- After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day)

**Example:**

```
Input: [1,2,3,0,2]
Output: 3 
Explanation: transactions = [buy, sell, cooldown, buy, sell]
```

## 我的解题思路

首先理清楚这个问题分解成子问题以后的状态

1. 今天卖而不是昨天卖，昨天什么都不做(sell) : $F[i] = F[i-1] + nums[i] - nums[i-1]$
2. 昨天卖，今天不卖（rest），也就是什么都不做或者说colddown $ F[i] = F[i-1] $
3. 昨天买，今天卖，前天是colddown $F[i] = F[i - 3] + nums[i] - nums[i-1]$

错误的！！！！！==错在第二步，没考虑到昨天卖，==

## 别人的，答案

 [视频讲解]](https://www.bilibili.com/video/av31578180?from=search&seid=2067286495683712471)

状态机里的三种状态，分别是`休息`(colddown)，`hold`(什么都不做)，`sell`（卖出）

 ![状态转移方程和状态机](http://zxi.mytechroad.com/blog/wp-content/uploads/2018/01/309-ep150.png) 



状态转移方程和状态机能对比起来，从状态图可以看出，`Rest`状态只能从 `sold`状态和`rest`状态变换过来，所以这时候rest的状态转移方程就为$rest[i] = max(rest[i - 1],sold[i-1])$ ,

这个状态机很重要！

