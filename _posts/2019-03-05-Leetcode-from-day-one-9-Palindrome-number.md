---
layout: post
title: '[LeetCode From Day One] - 9. Palindrome number'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
last_modified_at: 2019-03-05T10:04:37-05:00
---

本题是LeetCode简单难度第九题[回文数](https://leetcode.com/problems/palindrome-number/description/)，这里仍然先给出题目与相应的提示与要求。

> 给定一个整数x，判断其是否位回文数。  
> Example:   
> 	Input: 121  
> 	Output: true  
> 能否在不转为字符串的情况下实现（额外的存储空间）？  

鉴于[Reverse Integer](https://www.hytheory.com/algorithm/Leetcode-from-day-one-7-Reverse-Integer/)这篇文章中介绍了如何反转一个数组，这里很容易就能想到的方法就是比较反转前与反转后的数字。

```java
public class Solution
{
    public bool IsPalindrome(int x)
    {
        if (x < 0)
            return false;
        else if (x == 0)
            return true;
        else
        {
            int tmp = x;
            int reverse = 0;
            while (x != 0)
            {
                reverse = reverse * 10 + x % 10;
                x = x / 10;
            }
            if (reverse == tmp)
                return true;
            else
                return false;
        }
    }
}
```

然而，文章中也提到了反转后溢出的问题。所以本题应该是一个更通用的解法。

# Solution

首先我们可以发现一个负数不可能是一个回文数，因为负号的存在。0是一个回文数。对于一个正整数，我们可以根据回文数定义（正反排列相等），每次比较头部与尾部数字是否相等。
直接上代码：

```java
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0) {
            return false;
        }
        else if(x == 0) {
            return true;
        }
        int len = 1;
        while(x/len >= 10) {
            len *= 10;
        }

        while( x!=0 ) {
            //left == right
            if(x/len != x%10) {
                return false;
            }
            //掐头去尾
            x = (x % len) / 10;
            len /= 100;
        }
        return true;
    }
}
```

第一个while循环计算整数x的长度，若x=121，则len=100。计算len的作用是为了每次能取出最左边的数字，只需要用x对len取整即可。第二个while中的if条件就是在判断头部跟尾部是否相等，若相等，则`x = (x % len) / 10;`得到掐头去尾后的x。

