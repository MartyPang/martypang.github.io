---
layout: post
title: '[LeetCode From Day One] - 7. Reverse Integer'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Overflow
last_modified_at: 2019-03-04T16:10:34-05:00
---


# Problem

先给出LeetCode上的原题 [Reverse Integer](https://leetcode.com/problems/reverse-integer/description/)。

> 给定一个32位带符号的整数x，给出其反转的结果。
> Example:
> 	Input: 123
> 	Output: 321
> 提示：假设机器为32位，能存储的32位带符号整数的范围在: $[-2^{31}, 2^{31}-1]$($[-2147483648, 2147483647]$)。如果反转的整数溢出，则函数返回0。

# Naive Solution

这道题初看是在考察取模与取整的基础知识，可以很容易给出一个最基本的解决方案。思路可以这么理解，即不断得将原数的最后一位插入到反转数字的末尾。下面给出Java代码：

```java
class Solution {
    public int reverse(int x) {
        int reverse = 0;
        while(x != 0) {
            int last = x % 10;
            x /= 10;
            reverse = reverse*10 + last;
        }
        return reverse;
    }
}
```

但事实上，本题的关键是在如何检测整数溢出。存在一些情况，使得反转后的数字超过机器所能表示的整数范围。比如$-2147483648$反转后得到$-8463847412$，超出了32位有符号int能表示的最小数。

# 运算前检测溢出

检测整数是否溢出通常采用运算前检测的方法，即使用`MAX_VALUE`或者`MIN_VALUE`做一次逆转运算，判断该运算会不会产生一个溢出的整数。在计算新的`reverse`之前判断运算的合法性。给出AC的Java代码：

```java
class Solution {
    public int reverse(int x) {
        int reverse = 0;
        while(x != 0) {
            //运算前先判断是否会越界
            int last = x % 10;
            x /= 10;
            //reverse*10或者reverse*10+last溢出上界
            if(Integer.MAX_VALUE/10 < reverse || (Integer.MAX_VALUE/10 == reverse && 7 < last)) {
                return 0;
            }
            //reverse*10或者reverse*10+last溢出下界
            else if(Integer.MIN_VALUE/10 > reverse || (Integer.MIN_VALUE/10 == reverse && -8 > last)) {
                return 0;
            }
            reverse = reverse*10 + last;
        }
        return reverse;
    }
}
```

以第一个if语句，举个简单的例子。假设最大的int值为97，我们有$reverse=9$，$last=8$，则第一个条件`Integer.MAX_VALUE/10 < reverse`不满足，因为$97/10==9$ ，并且$7<last$。也就是说如果reverse再做一次运算即会溢出，最终函数返回0。

# 背后的知识

借此总结归纳一下32位机器各C++数据类型的表示范围。

| 类型 | 字节数 |  数值范围  |
| :--: | :----: | :--------: |
| bool |   1    | true/false |
| char/__int8 |   1    |    $[-2^7, 2^7-1]$/$[-128, 127]$    |
| unsigned char | 1 | $[0, 2^8-1]$/$[0, 255]$ |
| short/__int16 | 2 | $[-2^{15}, 2^{15}-1]$/$[-32768, 32767]$ |
| unsigned short | 2 | $[0, 2^{16}-1]$/$[0, 65535]$ |
| int/__int32 | 4 | $[-2^{31}, 2^{31}-1]$/$[-2147483648, 2147483647]$ |
| unsigned int | 4 | $[0, 2^{32}-1]$ |
| long | 4 | $[-2^{31}, 2^{31}-1]$/$[-2147483648, 2147483647]$ |
| unsigned long | 4 | $[0, 2^{32}-1]$ |
| float | 4 | $[-(2-2^{-23}) \times 2^{127}, -1.0 \times 2^{-126}]$ ,  $0$ , $[1.0 \times 2^{-126}, -(2-2^{-23}) \times 2^{127}]$ |
| double | 8 | $[-(2-2^{-52}) \times 2^{1023}, -1.0 \times 2^{-1022}]$, $0$, $[1.0 \times 2^{-1022}, -(2-2^{-52}) \times 2^{1023}]$ |

部分资料参考：

[IEEE Standard for Floating-Point Arithmetic (IEEE 754).](https://en.wikipedia.org/wiki/IEEE_754)