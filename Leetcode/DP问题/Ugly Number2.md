# Ugly NumberII

## 问题描述

Write a program to find the `n`-th ugly number.

Ugly numbers are **positive numbers** whose prime factors only include `2, 3, 5`. 

**Example:**

```
Input: n = 10
Output: 12
Explanation: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 is the sequence of the first 10 ugly numbers.
```

**Note:** 

1. `1` is typically treated as an ugly number.
2. `n` **does not exceed 1690**.

## 解决方法

首先参考 `[ugly number](https://leetcode.com/problems/ugly-number/)` 

isUglyNumber 

```
public boolean isUgly(int num) {
    if(num==1) return true;
    if(num==0) return false;
	while(num%2==0) num=num/1;
	while(num%3==0) num=num/3;
	while(num%5==0) num=num/5;
    return num==1;
}
```

通过上面可以知道，一个数如果时ugly，number,那么必须要时 2，3，5这三个数的乘积。不能包含其他质数，最后化解的结果肯定只有这三个数

1. 1,2,3,4,5 这5个是前5个ugly number
2. 第六个是 6 = 2 * 2 * 2 = 4 * 2
3. 第7个是 8 = 2 * 3
4. 第8个是  9 = 3 * 3 
5. 第9个是 10 = 2 * 5
6. 第 10个是 12 = 2 * 2  * 3 = 6 * 2 
7. 第11个是 15 = 3 * 5 =15
8. 第12个是  16 = 2 * 8 = 16
9. 第13个是  18 = 9 * 2  = 18
10. 第14个是  20 = 10 * 2 = 20

### 状态转移方程推演

3个变量 i = 2, j =3, k =5

1. 全部乘以2 ，取最小值 有 ： i = 4 即为下一个值，现在有  i = 4. j = 3 , k = 5
2. 全部乘 2，取最小值，有 j * 2 = 6,现在有  i = 4, j = 3 , k = 5
3. i = 6 , j = 3, k = 5 



### 解题方法（别人的！）

```
1 Declare an array for ugly numbers:  ugly[n]
2 Initialize first ugly no:  ugly[0] = 1
3 Initialize three array index variables i2, i3, i5 to point to 
   1st element of the ugly array: 
        i2 = i3 = i5 =0; 
4 Initialize 3 choices for the next ugly no:
         next_mulitple_of_2 = ugly[i2]*2;
         next_mulitple_of_3 = ugly[i3]*3
         next_mulitple_of_5 = ugly[i5]*5;
5 Now go in a loop to fill all ugly numbers till 150:
For (i = 1; i < 150; i++ ) 
{
    /* These small steps are not optimized for good 
      readability. Will optimize them in C program */
    next_ugly_no  = Min(next_mulitple_of_2,
                        next_mulitple_of_3,
                        next_mulitple_of_5); 

    ugly[i] =  next_ugly_no       

    if (next_ugly_no  == next_mulitple_of_2) 
    {             
        i2 = i2 + 1;        
        next_mulitple_of_2 = ugly[i2]*2;
    } 
    if (next_ugly_no  == next_mulitple_of_3) 
    {             
        i3 = i3 + 1;        
        next_mulitple_of_3 = ugly[i3]*3;
     }            
     if (next_ugly_no  == next_mulitple_of_5)
     {    
        i5 = i5 + 1;        
        next_mulitple_of_5 = ugly[i5]*5;
     } 
     
}/* end of for loop */ 
6.return next_ugly_no
```