#  Best Time to Buy and Sell Stock II 

## 问题描述

Say you have an array for which the *i*th element is the price of a given stock on day *i*.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times).

**Note:** You may not engage in multiple transactions at the same time (i.e., you must sell the stock before you buy again).

**Example 1:**

```
Input: [7,1,5,3,6,4]
Output: 7
Explanation: Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4.
             Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.
```

**Example 2:**

```
Input: [1,2,3,4,5]
Output: 4
Explanation: Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
             Note that you cannot buy on day 1, buy on day 2 and sell them later, as you are
             engaging multiple transactions at the same time. You must sell before buying again.
```

**Example 3:**

```
Input: [7,6,4,3,1]
Output: 0
Explanation: In this case, no transaction is done, i.e. max profit = 0.
```

## 解题方法

DP问题，设$F[i]$ 为第i天卖点股票获得的最大值，那么有

$F[i] = Max(F[i])$

==啊啊啊啊！！！ 把问题想复杂了! 只要前一个值比当前值小，那么就可以进行操作交易==

```java
public class Solution {
    public int maxProfit(int[] prices) {
       if (prices.length < 1)
            return 0;
        int sumProfit = 0;
        int buyIndex = prices[0]; //assume buy
        int profit = 0;
        for (int index = 1; index < prices.length;index++){
            //可以求得局部最优解,得到的profit可以加入sumProfit中。并买下 下一个Index的商品
            if (prices[index] - buyIndex > 0){
                sumProfit += prices[index] - buyIndex;
                buyIndex = prices[index]; //买入下个Index 商品
                continue;
            }
            if (buyIndex > prices[index]) //convert buy
                buyIndex = prices[index];
        }
        return sumProfit;
    }
}
```

