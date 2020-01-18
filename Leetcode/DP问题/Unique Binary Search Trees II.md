#   Unique Binary Search Trees II 

## 问题描述

Given an integer *n*, generate all structurally unique **BST's** (binary search trees) that store values 1 ... *n*.

**Example:**

```
Input: 3
Output:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
Explanation:
The above output corresponds to the 5 unique BST's shown below:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

## 我的解题思路

n 输入，r 输出

1. n = 1， r = 1 
2. n = 2 ,   r = 2 : 2作为 root,那么child只有1，结果自然是2
3. n = 3,    r = 5 ： 
4. n = 4,    r = 14
5. n = 5,    r = 42

考虑，二叉树的root确定，二叉树限制的n确定. root < n 。大于root的 在右子树，小于root的在左子树。以确定的root的完全二叉树应该一共有 **$F(n - root)  * F(root - 1)$  ** 其实这个时候就确定状态转移方程了。再考虑初始化情况，当 n = 1时，结果等于 1 ，当 n = 2 时，结果等于2 。所以应该有$F(0) = 1  ， F(1) = 1$.

由此我们得出状态转移方程和初始化情况：

$f(n) = \sum_{i=1}^n f(n - i) * f(i - 1)$ ,其中$f(0) = 1,f(1) = 1$



```伪代码
for i < n: //以i 为root
  for j <= i: 
    dp[i+1] += dp[i - j] * dp[j-1]
dp[n]
```



code :

```go

func numTrees(n int) int {
    if n <= 1{
        return 1
    }
    dp := make([]int,n+1)
    dp[0] = 1
    dp[1] = 1
    for i := 1;i < n;i++{
        //
        for root := i;root >= 0;root--{          
            dp[i+1] += dp[root] * dp[i - root]    
        }
    }
    return dp[n]
}
```

