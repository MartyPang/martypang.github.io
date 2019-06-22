---
layout: post
title: '[LeetCode From Day One] - Reverse Linked List II'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Linked List
last_modified_at: 2019-06-21T14:28:21-05:00
---

之前的一篇介绍单链表数据结构文章中，讲过反转链表的迭代算法和递归算法。递归的基本思路是利用递归栈走到链表的尾部，从尾部开始向前反转。本文完整代码见[GitHub](<https://github.com/MartyPang/DataStructures/blob/master/src/main/java/ecnu/dase/list/MyLinkedList.java>)。

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

若`head.next == null`，说明该head为反转链表的新头部。该递归算法最关键的两行代码在于：

```java
head.next.next = head;
head.next = null;
```

这两行代码反转指针，并把当前head的next置为空。

本文介绍上述基本链表反转问题的两个扩展问题，分别是反转链表的前n个节点，以及反转链表从位置`m`到位置`n`的节点。

# N reverse

反转前n个节点与反转整个链表唯一不同的点在于到达第n个节点后，需要记录节点n的next节点，否则反转后，链表就断了。反转整个链表之所以不用记录successor，最后一个节点的next为null。

```java
class Solution {
    //successor记录最后一个反转节点的next
    ListNode successor = null;
    public ListNode nreverse(ListNode head, int n) {
        if(n == 1) {
            successor = head.next;
            return head;
        }
        ListNode newHead = nreverse(head.next, n-1);
        head.next.next = head;
        head.next = successor;
        return newHead;
    }
}
```

# [M, N] reverse

在上一个问题的基础上，再增加一个左边界，变成反转区间`[m,n]`之间的节点。这个问题可以转化为反转链表的前n个节点问题（[LeetCode中等难度92题](https://leetcode.com/problems/reverse-linked-list-ii)）。我们只需要找到第m个节点作为`nreverse`的输入，第m节点之后的`n-m+1`个节点即可。

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        if(m == 1) {
            return nreverse(head, n);
        }
        head.next = reverseBetween(head.next, m-1, n-1);
        return head;
    }

}
```

**迭代解法**

也可使用迭代的方式来做，基本思路就是，首先要找到第m个位置的节点，并且记录m-1节点。从m节点开始，反转指针直至第n个节点。反转第n个节点之前，同样要记录n+1节点，否则链表会断。反转指针需要借助第三个临时指针`tmp`。

LeetCode上提供该种解法很好的示意图来解释算法。

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        if(m == n || head == null || head.next == null) return head;
        ListNode cur = head, prev = null;
        while(m > 1){
            prev = cur;
            cur = cur.next;
            --m;
            --n;
        }
        // con为第m-1个节点，tail为第m个节点，也是反转链表的尾部
        ListNode con = prev, tail = cur;
        ListNode tmp = null;
        while(n > 0) {
            //swap
            tmp = cur.next;
            cur.next = prev;
            prev = cur;
            cur = tmp;
            --n;
        }
        if(con != null) con.next = prev;
        else head = prev;
        tail.next = cur;
        return head;
    }
}
```