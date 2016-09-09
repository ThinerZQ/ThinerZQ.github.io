title: 172. Factorial Trailing Zeroes
date: 2016-09-09 13:49:06
tags: [leetcode,math]
categories: [leetcode]
---
# 题目描述

Given an integer n, return the number of trailing zeroes in n!.

Note: Your solution should be in logarithmic time complexity.

<!-- more -->
# 解题
## 解法一
煞笔的纯粹计算出n的阶乘，然后取出最后的0的个数，这样会导致n太大溢出。
## 解法二
n! 除了1以外必然是合数，合数可以做质因数分解，n!分解质因数的表达式如下：
>n! = 2^x * 3^y * 5^z * ...

一个数末尾0的个数，取决于它的质因子中2和5的个数的最小值，对于本题就是min(x,z)=z

证明:
证明：
<pre>
对于阶乘而言，也就是1*2*3*...
[n/k]代表1~n中能被k整除的个数
那么很显然
[n/2] > [n/5] (左边是逢2增1，右边是逢5增1)
[n/2^2] > [n/5^2](左边是逢4增1，右边是逢25增1)
……
[n/2^p] > [n/5^p](左边是逢2^p增1，右边是逢5^p增1)
随着幂次p的上升，出现2^p的概率会远大于出现5^p的概率。
因此左边的加和一定大于右边的加和，也就是n!质因数分解中，2的次幂一定大于5的次幂
</pre>

## 代码
```java
public int zeroes(int n) {
       int sum=0;
       int count =0;
       int k=n;
       while (n>0){
           n=n/5;
           count++;
       }
       count--;
       while (count>0){
           sum+= (k/(Math.pow(5,count--)));
       }
       return sum;
   }
```
