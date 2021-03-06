---
layout: post
title: 'Swap two numbers'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - Swap
last_modified_at: 2019-04-22T13:57:31-05:00
---

在上一篇介绍排序算法的文章中经常会用到交换两数，可用的方法很多，有使用临时内存的交换方式与不使用临时内存的方式。本文简单总结三种以及相对应的注意点。

## 临时变量

最常见的交换两数方法借助一个临时变量实现。

```java
int tmp = a;
a = b;
b = tmp;
```

Java由于对普通类型的变量不支持**引用传递**，没办法像C/C++一样单独写一个swap函数。C/C++函数可以传递引用传递指针。

```c++
void swap(int &a, int &b) {
    int tmp = a;
    a = b;
    b = tmp;
}
```

## 算术运算

利用加减或者乘除或者位运算的运行性质完成两数的交换，这种方式不开辟新的内存，效率较高。

### 加减/乘除

直接看代码。

```java
a = a + b; //此时a为a+b
b = a - b; //b为a+b-b
a = a - b; //a为a+b-a
```

同理乘除也可。

```java
a = a * b;
b = a / b;
a = a / b;
```

### 位运算

利用异或的性质，同样可以实现不使用临时变量交换两数。我们知道**x ^ x = 0**，**x ^ 0 = x**，再结合异或的交换律可以得出**x ^ y ^ x = x ^ x ^ y = y**。

```java
a ^= b;
b ^= a;
a ^= b;
```

两种不使用临时变量的交换方式在使用时要注意，`a`，`b`两数的地址不能相同，否则值会变为0。

