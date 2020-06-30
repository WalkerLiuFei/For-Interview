## 解体思路

```
dp[i][j]`: the longest palindromic subsequence's length of substring(i, j), here i, j represent left, right indexes in the string
`State transition`:
`dp[i][j] = dp[i+1][j-1] + 2` if s.charAt(i) == s.charAt(j)
otherwise, `dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1])`
`Initialization`: `dp[i][i] = 1
```