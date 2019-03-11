---
layout: post
title: '[LeetCode From Day One] - Addition and Subtraction'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
last_modified_at: 2019-03-11T10:18:09-05:00
---

周末刷了两个简单的加法题，[66题整形数组加1](https://leetcode.com/problems/plus-one/description/)和[67题二进制数的加法](https://leetcode.com/problems/add-binary/description/)。题目解法比较类似，注意点都是要考虑进位，故在此做一个简单总结。

# 整型数组加1

> 给定一个非空的整形数组，每个元素代表0-9，返回该数组加1后的结果。假设数组不包含任何前导0。
> Example: 
&nbsp; &nbsp; Input: [1,2,3]  
&nbsp; &nbsp; Output: [1,2,4]  

解法也很简单，从数组最后一位开始往前遍历，最后一位加1，其余每位加上上一位的进位。跳出遍历循环后如果进位仍旧为1，则返回结果多加一位。

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int carry = 0;
        if(digits[digits.length-1] + 1 == 10) {
            digits[digits.length-1] = 0;
            carry = 1;
        }
        else {
            digits[digits.length-1] += 1;
        }
        for(int i = digits.length-2; i > -1; --i) {
            if(digits[i] + carry == 10) {
                digits[i] = 0;
                carry = 1;
            }
            else {
                digits[i] += carry;
                carry = 0;
                break;
            }
        }
        if(carry == 1) {
            int[] result = new int[digits.length + 1];
            result[0] = 1;
            for(int i = 1; i < digits.length + 1; ++i) {
                result[i] = digits[i-1];
            }
            return result;
        }
        else {
            return digits;
        }
    }
}
```

# 二进制数的加法

> 给定两个字符串类型的二进制数，返回它们的和（二进制字符串返回）。输入字符串非空并且只包含0和1。
> Example: 
&nbsp; &nbsp; Input: a="11", b="1"  
&nbsp; &nbsp; Output: "100"

与上一题类似的做法，两个二进制数末尾对齐，从最后一位开始做加法，满2进位。有个小技巧，char类型的数字如何转int类型，根据`ASCII码`，只需要`char - '0'`即可。

```java
class Solution {
    public String addBinary(String a, String b) {
        String result = "";
        int i = a.length()-1, j = b.length() - 1;
        int carry = 0;
        while(i >= 0 || j >= 0) {
            if(i >= 0) {
                carry += a.charAt(i) - '0';    
            }
            if(j>= 0) {
                carry += b.charAt(j) - '0';
            }
            result = carry % 2 + result;
            carry = carry / 2; 
            --i;
            --j;
        }
        if(carry != 0) {
            result = 1 + result;
        }
        return result;
    }
}
```

计算carry时，对2取模即可。同样跳出循环之后要判断carry是否为1。