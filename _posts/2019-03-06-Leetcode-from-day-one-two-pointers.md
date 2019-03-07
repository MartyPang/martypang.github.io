---
layout: post
title: '[LeetCode From Day One] - Two Pointers'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Pointer
last_modified_at: 2019-03-06T15:16:21-05:00
---

本文介绍如何使用双指针（Two Pointers）解决LeetCode简单难度[21题归并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/description/)，[26题有序数组去重](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)与[27题删除数组中的指定元素](https://leetcode.com/problems/remove-element/description/)。

第一题遍历两个链表，后两道题均要求不开辟额外的存储空间，就地（in-place）修改原数组。容易想到经常用于遍历数组，或指向不同元素或一快一慢的双指针，协同完成任务。

# 归并有序链表

> 归并两个有序链表，返回归并后的结果。  
> Example:  
&nbsp; &nbsp; Input: 1->2->4, 1->3->4  
&nbsp; &nbsp; Output: 1->1->2->3->4->4    

我们可以初始化两个指针，分别遍历两个链表。每次循环，值小的指针往前走一步，直到两个指针都走到链表的末尾。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode head = new ListNode(-1);
        ListNode tmp = head;
        while(l1 != null || l2 != null) {
            if(l1 == null) {
                tmp.next = l2;
                l2 = l2.next;
                tmp = tmp.next;
            }
            else if(l2 == null) {
                tmp.next = l1;
                l1 = l1.next;
                tmp = tmp.next;
            }
            //p1 != null && p2 != null
            else {
                if(l1.val <= l2.val) {
                    tmp.next = l1;
                    l1 = l1.next;
                    tmp = tmp.next;
                }
                else {
                    tmp.next = l2;
                    l2 = l2.next;
                    tmp = tmp.next;
                }
            }
        }
        return head.next;
    }
}
```

# 有序数组去重

> 给定一个排好序的数组，去除重复元素并返回剩余元素的个数。  
> Example:  
&nbsp; &nbsp; 给定nums = [0,0,1,1,1,2,2,3,3,4]，函数需返回长度5，且数组的前五个元素分别为0，1，2，3和4。
> Note: 不要分配额外的数组，直接修改传入的参数。


本题同样使用双指针来完成。分配两个指针`i`和`j`，`i`遍历数组`nums`，`j`指向最新的无重复元素。代码很简短，如果`i`指向的值与`j`指向的值相同，`i`走一步，`j`原地不动。否则，`j`走一步并把`nums[j]`置为`nums[i]`。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length == 0) return 0;
        int i = 0;
        int j = 0;
        while(i < nums.length) {
            if(nums[i] == nums[j]) {
                ++i;
            }
            else {
                nums[++j] = nums[i];
            }
        }
        return j+1;
    }
}
```

# 删除数组的指定元素

> 给定一个数组`nums`和待删除的值`val`，删除数组中所有`val`，返回剩余数组的长度。
> Example:  
&nbsp; &nbsp; 给定nums = [3,2,2,3]，val = 3，函数返回2。
> 与26题相同的要求，不要分配额外的数组，直接修改传入的参数。


本题解法与26题类似，同样使用双指针进行处理。这里不再赘述，直接贴代码。

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if(nums.length == 0) return 0;
        int i = 0;
        int j = 0;
        while(i < nums.length) {
            if(nums[i] != val) {
                nums[j++] = nums[i];
            }
            ++i;
        }
        return j;
    }
}
```

# 其他使用双指针的场景

回文，反转字符串，两数平方和等。