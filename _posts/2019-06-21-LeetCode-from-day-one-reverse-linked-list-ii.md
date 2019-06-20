---
layout: post
title: '[LeetCode From Day One] - Reverse Linked List II'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Linked List
last_modified_at: 2019-06-20T14:28:21-05:00
---

之前的一篇介绍单链表数据结构文章中，讲过反转链表的迭代算法和递归算法。递归的基本思路是利用递归栈走到链表的尾部，从尾部开始向前反转。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null || head.next == null) return head;
        ListNode newHead = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;
    }
}
```

本文介绍上述基本链表反转问题的两个扩展问题，分别是反转链表的前n个节点，以及反转链表从位置`m`到位置`n`的节点。

# N reverse

```java

```

# [M, N] reverse