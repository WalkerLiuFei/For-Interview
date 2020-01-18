#  Longest Palindromic Substring 

## 问题描述

 https://leetcode.com/problems/longest-palindromic-substring/ 

Given a string **s**, find the longest palindromic substring in **s**. You may assume that the maximum length of **s** is 1000.



**Example 1:**

```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```

**Example 2:**

```
Input: "cbbd"
Output: "bb"
```



最长回文字符串

**一共提交了6次才通过**

## 解题思路





```go

```

这种

首先考虑回文的两种形式

1:baab型，即他们通过中间对称轴对称

2:bcacb型，即他们通过中间的元素对称

这个问题肯定是可以通过遍历求解的，即通过查询所有的字符串然后找到结果，这种时间复杂度为$O(N^2)$

伪代码:

```
result := s[0]
i = 1
for i++ < n -1:
  var j int 
  var k int
  if s[i] == s[i+1]
    j = i
    k = i+1
  if s[i-1] == s[i+1]
    j = i-1
    k = i+1
  for ;j >= 0 && k < len(n);j--,k++:
	if s[j] != s[k]
       break
    if len(result) <  k - j + 1:
       result := s[j:k+1]
```

**经过提交测试伪代码没有发现的问题**

1. “aa”的情况
2. “aab”的情况
3. “abba”的情况



正确的代码应该是



```go
func longestPalindrome(s string) string {
    if len(s) <= 1{
        return s
    }
    result := s[:1]
    
    if s[0] == s[1]{
        result =  s[:2] 
    }
    
    for i :=1; i < len(s) -1;i++{
        var j int 
        var k int
        if s[i] == s[i+1] {
            j = i
            k = i+1
            for ;j >= 0 && k < len(s);{
                if s[j] != s[k]{
                 break
                }
                if k -j +1 > len(result){
                     result =s[j:k+1]
                }
                j--
                k++
            }
        }
        if  s[i-1] == s[i+1]{
            j = i-1
            k = i+1
            for ;j >= 0 && k < len(s);{
               if s[j] != s[k]{
                  break
               }
               if k -j +1 > len(result){
                  result =s[j:k+1]
               }
               j--
               k++
            }
        }
        
    }
    return result
}
```

这种方式感觉很粗暴，但是好像没有更好的解决方式，感觉和DP也挂不上钩

