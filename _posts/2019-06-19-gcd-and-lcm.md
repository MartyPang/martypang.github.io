---
layout: post
title: 'GCD & LCM'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - GCD
  - LCM
last_modified_at: 2019-06-19T14:28:21-05:00
---

本文总结最大公约数（GCD），最小公倍数（LCM）相关的性质，求解算法以及算法应用。两个或多个不全为0的数的最大公约数表示能整除每个数的最大整数，例如12与8的最大公约数为4。而两个数的最小公倍数，`LCM(m,n)`，表示最小的能被`m`和`n`同时整除的正整数，其中`m`，`n`均不为0。

* 目录
{:toc}

# 性质

$gcd(m,n)=gcd(n,m)$

$gcd(m,0)=m$

$gcd(0,n)=n$

若`m`与`n`互质，则$gcd(m,n)=1$。根据互质定义，两数除1以外没有别的公约数。

最有用的一条性质应该是：$$lcm(m,n) * gcd(m,n)=m * n$$，即两个数的最小公倍数与最大公约数之积为两数之积。因为`m/gcd(m,n)`与`n/gcd(m,n)`两数互质，即$gcd(m/gcd(m,n),n/gcd(m,n))=1$。又$lcm(m,n)=gcd(m,n) * (a/gcd(m,n)) * (b/gcd(m,n))$，得到$lcm(m,n)=(a * b)/gcd(m,n)$。

# 最大公约数 - GCD

求两个数最大公约数的常用算法，欧几里得算法（辗转相除法），基于这样一条性质$gcd(m,n)=gcd(n,m \% n)$。

## 循环写法

```java
int gcd(int m, int n) {
  int tmp;
  while(n != 0) {
    tmp = m%n;
    m=n;
    n=r;
  }
  return a;
}
```

## 递归写法

递归写法就一行代码。

```java
int gcd(int m, int n) {
  return n==0?m:gcd(n, m%n);
}
```

## n个数的最大公约数

$gcd(n_1,n_2,n_3...)=gcd(gcd(n_1,n_2),n_3...)$

```java
int ngcd(int[] a) {
  int ans = a[0];
  for(int i = 1; i < a.length; ++i) {
    ans = gcd(ans, a[i]);
  }
  return ans;
}
```

# 最小公倍数 - LCM

最小公倍数可以利用性质$lcm(m,n) * gcd(m,n)=m * n$转化为求两个数的GCD。

```java
int lcm(int m, int n) {
  return m * n / gcd(m, n);
}
```



# X of a Kind in a Deck of Cards

本题为LeetCode[简单难度914题](https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/)。题目要求输出能否找到一个`X>=2`，将一堆排分为几堆：
- 每堆牌有X张；
- 每堆牌牌面一致；

> Input: [1,2,3,4,4,3,2,1]
> Output: true
> Explaination: Possible partition [1,1] [2,2] [3,3] [4,4]

本题只需一趟遍历数组，得到每个数出现的次数，求出所有次数的最大公约数，即为X。我们使用hashmap存储数字出现的次数。

```java
class Solution {
    public boolean hasGroupsSizeX(int[] deck) {
        if(deck.length < 2) return false;
        Map<Integer, Integer> map = new HashMap<>();
        for(int d : deck) map.put(d, map.getOrDefault(d, 0)+1);
        List<Integer> list = new ArrayList<>(map.values());
        int m = list.get(0);
        for(int i = 1; i < list.size(); ++i) {
            m = gcd(m, list.get(i));
            if(m < 2) return false;
        }
        return true;
    }

    private int gcd(int m, int n) {
        if(n == 0) return m;
        else return gcd(n, m%n);
    }
}
```
