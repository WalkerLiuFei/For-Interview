# Decode ways

## 问题描述

A message containing letters from `A-Z` is being encoded to numbers using the following mapping:

```
'A' -> 1
'B' -> 2
...
'Z' -> 26
```

Given a **non-empty** string containing only digits, determine the total number of ways to decode it.

**Example 1:**

```
Input: "12"
Output: 2
Explanation: It could be decoded as "AB" (1 2) or "L" (12).
```

**Example 2:**

```
Input: "226"
Output: 3
Explanation: It could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).
```

## 解题过程

这个问题是困扰我很久的问题，应该是个实打实的DP问题。应该尝试了10几次提交了

## 我的解题思考过程

### 基本的思路

这种应该就是典型的DP问题，

首先考虑一个最简单例子 ： 111111

1. 1 -> A   =1 
2. 11 -> AA,K = 2
3. 111 -> AAA,AK,KA = 3
4. 1111 -> AAAA,KAA,AKA,AAK,KK = 5
5. 11111 > AAAAA,KAAAA,AKAA,AAKA,AAAK,KKA,KAK,KKA = 8
6.  111111 > ....... =  13
7.  1111111 > ......... = 21
8. 11111111 >  ........... = 21 + 13 = 34

**可以看出这其实是一个组合问题，即 斐波那契数列 ： F[0] = 0 , F[1] = 1,  F[i] = F[ i - 1] + F[ i -2 ]** 

再考虑一个例子：123456。

那么它的可能性解应该是

1. 1 --> A ,结果是1
2. 12 --> “AB”,“L”，结果是 step.1 + 1 = 2
3. 123 --> “ABC”,“LC”,“AW” 即 step.2 + 1 = 3
4. 1234 ---> “ABCD”,“LCD”,“AWD”,“LC” ,即step.3  + 0 =3
5. 12345 > ABCDE, LCDE,AWDE ，即step.4 + 0 = 3
6. 123456 > ABCDEF,LCDEF,AWDEF = step5 + 0 

**可以看出，放后续的数字过大无法组成有效字母时，其保持不变**

再考虑一个例子

1234111

1. 1 --> A ,结果是1
2. 12 --> “AB”,“L”，结果是 step.1 + 1 = 2
3. 123 --> “ABC”,“LC”,“AW” 即 step.2 + 1 = 3
4. 1234 ---> “ABCD”,“LCD”,“AWD”,“LC” ,即step.3  + 0 =3
5. 12341 > ABCDA,LCDA,AWDA  ，step.4 + 0 = 3
6. 123411 > ABCDAA,LCDAA,AWDAA,   ABCDK,LCDK,AWDK = 3  + 3
7.  1234111 > ABCDAAA,LCDAAA,AWDAAA,   ABCDK,LCDK,AWDK = 6 + 3 = 9

**可以看出，放后续的数字过大无法组成有效字母时，其保持不变,但当后续的字符重新可以组成字母时其保持回斐波那契数列状态**

### 特殊情况

==特殊情况，即包含0,==

1. **如果0前面的数字大于2 或者等于0那么这个string肯定是没办法decode了，因为没有那个A-Z的字符可以decode “00”,例如 1001这种**
2. 但是如果只包含一个零，例如 ：120

现在我们给这个“12012” 进行decode

1. 1 --> A ,结果是1
2. 12 --> “AB”,“L”，结果是 step.1 + 1 = 2
3. 120 --> “AT” ,结果是 step.2 - 1 = 1 ==  step.1 
4. 1201--> “ATA”,结果是 step.3 + 0 = 1
5. 12012 --> “ATAB”,“ATL” 结果是 step4 + 1= 2

因为0把前面一个数字给占掉了，也就是无法使其创建其他组合，这样就造成, F[i] = F[i - 2]的情况,**但是要保证s[i] == 1 或者s[i] == 2**，不然的话，是不行的。

### 大概结论

那么大概可以给这个列一个状态转移方程了：

if 

F[i] = F[i - 1]

伪代码

```
dp = int[len(s)+1] #长度需要是s的长度 + 1
dp[0] = 0
dp[1] = 1
for i < len(s):
  if s[i] == '0' and (s[i] < '1' or s[i] > '2') :
    return 0
  if s[i] == '0': 
     dp[i + 1] = dp[i - 1]
  else if  "10" < s[i-1 : i+1] < "26":
     dp[i+1] = dp[i] + dp[i-1]
  else:
     dp[i+1] = dp[i]
```



go代码

```go
func numDecodings(s string) int {
    if len(s) < 1 || s[0] == '0' {
        return 0
    }
    dp := make([]int,len(s) + 1)
    dp[0] = 1
    dp[1] = 1
    for i := 1;i < len(s);i++{        
        s1 := s[i-1:i+1]
        if s[i] == '0'{
          if s[i - 1] == '2' || s[i - 1] == '1'{
               dp[i + 1] = dp[i - 1] 
          }else{
               return 0
          }
        }else if "10" < s1 && s1 <= "26"{
            dp[i + 1] = dp[i] + dp[i - 1]
        }else {
            dp[i + 1] = dp[i]
        }
    }

    return dp[len(s)]
}
```



