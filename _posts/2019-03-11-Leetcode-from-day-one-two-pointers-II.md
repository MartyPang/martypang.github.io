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

之前[一篇博文](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers/)用几道题目简单介绍过如何使用双指针（Two Pointers）——[21题归并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/description/)与[26题有序数组去重](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)。这次刷的两题恰好是21题与26题数据结构的互换，即[88题归并两个有序数组](https://leetcode.com/problems/merge-sorted-array/description/)和[83题有序链表去重](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/)。更新几个应用双指针的题，[141题检测链表中的环路](https://leetcode.com/problems/linked-list-cycle/description/)，[160题两个链表的交集](https://leetcode.com/problems/intersection-of-two-linked-lists/description/)与[167题Two sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

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
        ListNode slow = head, fast = head;
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

# Linked List Cycle

> 给定一个链表，返回其是否包含环路。
> Example:  
&nbsp; &nbsp; Input: head = [3,2,0,-4], pos = 1
&nbsp; &nbsp; Output: true
&nbsp; &nbsp; Explaination: `pos`表示链表尾部链接的节点，本例中链表尾部节点链接到第二个节点，形成环路。

本题的解题思路与现实生活中，两个人在环形的跑道上跑步的情况是一样的，跑的快的一定会套圈跑得慢的。这里应用一快一慢两个指针，如果快指针先到了`null`，说明跑道是直线的，不存在环路。如果快慢指针想在某个节点相遇了，说明链表一定存在环路。这里我们可以让快指针一次走两步。

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while(fast != null) {
            slow = slow.next;
            fast = fast.next;
            if(slow != null && fast != null && fast.next != null) {
                fast = fast.next;
                if(slow.val == fast.val) return true;
            }
            else break;

        }
        return false;
    }
}
```

# Intersection of Two Linked List

> 给定两个链表，返回链表交集的开始节点。
> Example:  
&nbsp; &nbsp; Input: intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3  
&nbsp; &nbsp; Output: 值位8的节点的引用

本题要求返回两个链表交集的初始节点，如下图所示：

![intersection](/images/20190311/intersection.png){: .align-center}

本题需要我们找到在两个链表遍历过程中，第一个相同的节点。考虑一种特殊情况，如果两个链表长度一致，那么使用`pA`与`pB`两个指针一次一步分别遍历两个链表，同时比较`pA`与`pB`，注意不是比较value。若循环走到链表末尾，表示`A`、`B`没有交集，返回`null`。再推广到一般情况，假设我们在第一趟遍历得到`A`、`B`链表的长度`lenA`与`lenB`，在第二趟遍历开始之前，更长的链表先走`abs(lenA-lenB)`步，这样我们就获得两个相同长度的链表，再重复特殊情况即可。代码如下：

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA == null || headB == null) return null;
        ListNode pa = headA, pb = headB;
        int lengtha = 1, lengthb = 1;
        while(pa.next != null) {
            ++lengtha;
            pa = pa.next;
        }
        while(pb.next != null) {
            ++lengthb;
            pb = pb.next;
        }
        if(pa != pb) return null;
        pa = headA;
        pb = headB;
        int diff = Math.abs(lengtha - lengthb);
        if(lengtha > lengthb) {
            while(diff-- > 0) {
                pa = pa.next;
            }
        }
        else {
            while(diff-- > 0) {
                pb = pb.next;
            }
        }
        while(pa != pb) {
            pa = pa.next;
            pb = pb.next;
        }
        return pa;
    }
}
```

看本题LeetCode上的Discussion发现大家的方法更巧妙啊，无需获取`A`、`B`链表长度，直接把`A`的尾部与`B`的头部相连，`B`的尾部与`A`的头部相连，这样指针`pa`与`pb`走到第一个汇合节点的“路程”是一样的。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA == null || headB == null) return null;
        ListNode pa = headA, pb = headB;
        while(pa != pb) {
            pa = pa == null ? headB : pa.next;
            pb = pb == null ? headA : pb.next;
        }
        return pa;
    }
}
```
第二趟遍历的循环跳出条件要么是两个同时走到`null`要么走到交汇节点，无需别的判断，并不会陷入死循环。

# Two Sum II

在LeetCode上刷的第一题就是[Two Sum](https://leetcode.com/problems/two-sum/description)题，这是第二个版本，把无须的输入数组改为排好序的数组。思路比较简单，一左一右两个指针相向而行。如果当前指向的两个数的和小于`target`，说明左指针指的值小了，往右走一个；反之，右指针往左走一个。如此往复，直到和等于target或者左右指针指向同一个值，程序退出。

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int[] res = new int[2];
        int left = 0, right = numbers.length-1;
        while(left != right) {
            if(numbers[left] + numbers[right] < target) ++left;
            else if(numbers[left] + numbers[right] > target) --right;
            else {
                res[0] = ++left;
                res[1] = ++right;
                break;
            }
        }
        return res;
    }
}
```

# 总结

碰上需要遍历的题，考虑使用双指针，但也不能死板的只知道从头到尾遍历，形式可以多样，正过来，倒过来，一个快，一个慢等等。