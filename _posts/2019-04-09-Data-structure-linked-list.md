---
layout: post
title: 'Singly Linked List'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Linked List
last_modified_at: 2019-04-09T10:33:37-05:00
---

链表是一种存储元素集合的常见数据结构。链表的每个节点由数据与指向下一个节点的指针组成，如下图所示。
![node](/images/20190409/node.png){:	.align-center}
链表中第一个节点称为`head`，最后一个节点指向`null`。关于链表的问题有：i) 插入；ii) 删除；iii) 反转；iiii) 组合；etc. 由于链表本身就是个递归结构，许多问题都可以用递归的方式解决。另外，链表的问题时常涉及遍历，之前介绍的[双指针II](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers-II/)就是一种常用的解法。以下内容涉及的链表结构定义如下：

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}
```

# Remove LinkedList Elements

LeetCode简单难度[203删除链表元素](https://leetcode.com/problems/remove-linked-list-elements/description/)要求删除链表中的指定节点，很基础很简答，只需使用一个指针遍历即可。时间复杂度$O(n)$，空间复杂度$O(1)$。

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        if(head == null) return head;
        ListNode p = head;
        while(p.next != null) {
            if(p.next.val == val) p.next = p.next.next;
            else p = p.next;
        }
        return head.val == val ? head.next : head;
    }
}
```

# Reverse LinkedList

LeetCode简单难度[206题反转链表](https://leetcode.com/problems/reverse-linked-list/description/)也是链表问题中比较常见的操作，solution有很多，比如生成一个新的链表，采用头插法不断将原链表中的节点插入到表头，这种算法空间开销$O(n)$。这里介绍两种space complexity为$O(1)$的迭代的算法和一种递归的方式。

## 迭代I

第一种迭代的方式的基本思想是在遍历链表的同时，将当前节点的`next`指针指向上一个节点。由于节点没有`prev`指针，所以需要定义额外的指针保存上一个节点的引用。另外每次迭代还需要保存下一个节点的引用，否则改变当前节点的`next`后就无法继续遍历下去。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode p = null;
        ListNode q = head;
        while(q != null) {
            ListNode next = q.next;
            q.next = p;
            p = q;
            q = next;
        }
        return p;
    }
}
```

算法的时间复杂度$O(n)$，空间复杂度$O(1)$。

## 迭代II

第二种方式是就地更新的方式，基本思想与头插法类似，不过不开辟新的链表。我们首先在`head`节点之前插入一个`dummy`节点。以`dummy->1->2->3->null`为例，说明in-place update的实现方式。
- `dummy->1->2->3->null`
- `dummy->2->1->3->null`
- `dummy->3->2->1->null`

从节点1开始遍历，算法将遍历过程的每个节点插入到dummy node后面。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null) return head;
        ListNode dummy = new ListNode(-1);
        dummy.next = head;
        ListNode p = head;
        ListNode q = head.next;
        while(q != null) {
            p.next = q.next;
            q.next = dummy.next;
            dummy.next = q;
            q = p.next;
        }
        return dummy.next;
    }
}
```

算法的时间复杂度$O(n)$，空间复杂度$O(1)$。

## 递归

递归的方式稍微巧妙一些，递归栈达到链表尾部再开始反转。在youtube上找到一个解释的很清楚的视频。

{% include responsive-embed url="https://www.youtube.com/embed/MRe3UsRadKw" ratio="16:9" %}

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

# Palindrome LinkedList

本题是LeetCode简单难度[234回文链表](https://leetcode.com/problems/palindrome-linked-list/description/)。题目要求写一个算法返回输入链表是否是回文的。

> 给定一个链表，判断其是否是回文的。
> Example:  
&nbsp; &nbsp; Input: 1->2->2->1  
&nbsp; &nbsp; Output: true  
> Follow up: 能否在$O(n)$时间以及$O(1)$空间内解决。

之前做过[回文整数](https://leetcode.com/problems/palindrome-number/description/)，[回文字符串](https://leetcode.com/problems/valid-palindrome/description/)，与这两题不同的是，单链表在不反转的情况下只能从头到尾遍历，无法利用类似双指针，分别从头尾向中间遍历。若可以使用额外空间，那么可新生成一个反转的链表，再比较两个链表，但是不符合空间开销$O(1)$的要求。本题的解法要换换思路。如果我们可以找到链表的中间元素，那么便可从中间开始向两头遍历，判断元素是否相等。如何找到链表中间元素有一个小技巧，使用两个指针，快指针速度是慢指针的两倍，当快指针达到末尾时，慢指针的位置就是链表的中间。为使得指针可以从中间向`head`遍历，慢指针在移动过程中还需要完成链表反转。算法实现如下。

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null) return true;
        ListNode p1 = head;
        ListNode p2 = p1.next;
        ListNode p3 = head;
        ListNode pre = p1;
        while(p3.next != null && p3.next.next != null) {
            //two moves
            p3 = p3.next.next;
            //reverse
            pre = p1;
            p1 = p2;
            p2 = p2.next;
            p1.next = pre;
        }
        //odd number of elements
        if(p3.next == null) p1 = p1.next;

        while(p2 != null) {
            if(p1.val != p2.val) return false;
            p1 = p1.next;
            p2 = p2.next;
        }
        return true;
    }
}
```
