---
layout: post
title: '[LeetCode From Day One] - Two Pointers II'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Pointer
last_modified_at: 2019-03-11T15:11:35-05:00
---

之前[一篇博文](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers/)用几道题目简单介绍过如何使用双指针（Two Pointers）——[21题归并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/description/)与[26题有序数组去重](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)。这次刷的两题恰好是21题与26题数据结构的互换，即[88题归并两个有序数组](https://leetcode.com/problems/merge-sorted-array/description/)和[83题有序链表去重](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/)。

# 归并两个有序数组

> 给定两个有序数组`nums1`和`nums2`，把`nums2`归并到`nums1`。假设`nums1`有足够的空间容纳。  
> Example:   
&nbsp; &nbsp; Input: nums1=[1,2,3,0,0,0], m=3; nums[2]=[2,5,6], n=3  
&nbsp; &nbsp; Output: [1,2,2,3,5,6]  

这个题目使用双指针分别遍历`nums1`和`nums2`。但是如果与21题一样，从头到尾遍历数组，因为不再额外申请空间，所以每次往`nums1`插入一个数x，需要`nums1`中比x大的元素均往后移动一位，效率不高。因此我们换一种思路，从尾到头遍历两个数组，每次选较大的元素插入`nums1`的末尾。

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m-1, j = n-1, p = m+n-1;
        while(i > -1 && j > -1) {
            if(nums1[i] > nums2[j]) {
                nums1[p--] = nums1[i--];
            }
            else {
                nums1[p--] = nums2[j--];
            }
        }
        while(j > -1) {
            nums1[p--] = nums2[j--];  
        }
    }
}
```

还有个比较巧妙的地方在于第二个`while`，因为是将`nums2`合并到`nums1`，如果跳出第一个循环时，`j`仍未遍历到`nums2`的头部，说明剩余未遍历元素比`nums1`的都小。而如果是`i`未走到`nums1`头部，则不做任何处理，因为已经是有序数组。算法时间复杂度$O(m+n)$。

# 有序链表去重

> 给定一个排好序的链表，删除所有的重复元素。  
> Example:   
&nbsp; &nbsp; Input: 1->1->2->3->3  
&nbsp; &nbsp; Output: 1->2->3  

本题的思路可以说与26题完全一致，使用一快一慢两个指针遍历链表，无非是下表访问与指针访问的区别。直接贴Java代码。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode  slow = head, fast = head;
        while(fast != null) {
            if(fast.val == slow.val) {
                fast = fast.next;
                if(fast == null) {
                    slow.next = fast;
                }
            }
            else {
                slow.next = fast;
                slow = fast;
            }
        }
        return head;
    }
}
```

# 总结

碰上需要遍历的题，考虑使用双指针，但也不能死板的只知道从头到尾遍历，形式可以多样，正过来，倒过来，一个快，一个慢等等。