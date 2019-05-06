---
layout: post
title: '[LeetCode From Day One] - Two Pointers K-diff in an Array'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Two Pointers
last_modified_at: 2019-05-06T15:20:31-05:00

---

本文是双指针系列的第三篇了，本题要比之前的[Two Pointers I](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers/)和[Two Pointers III](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers-II/)涉及的题目要难一点。本题是Amazon的笔试/面试题，要求在给定的数组中找到差的绝对值等于K的所有pair。

> 给定一个整数数组和一个整数k，要求找到所有的`(i,j)`对，它们的差的绝对值恰好为k。输出这样的pair的数量，注意要去重。
> Example：  
&nbsp; &nbsp; Input: [3,1,4,1,5]  
&nbsp; &nbsp; Output: 2
&nbsp; &nbsp; Expaination: 共有两个对的差的绝对值为k，`(1,3)`和`(3,5)`。虽然有两个1，但是题目要求unique。

首先能想到的方法是，对数组的元素做一个索引，然后遍历每个元素x，去索引中查找x+k的元素存在与否。这也是本题Discussion中vote数最多的[解法](https://leetcode.com/problems/k-diff-pairs-in-an-array/discuss/100098/Java-O(n)-solution-one-Hashmap-easy-to-understand)，使用HashMap，对数组与map的key分别做一次遍历，时间复杂度为$O(n)$，有兴趣的可以去看看代码。这种方式在第一趟遍历简历索引时就已经对元素去重了。

这里给出一种使用双指针的解法。基本的算法思想：两个指针遍历一个排好序的数组，快指针去找比慢指针大k的元素，慢指针遍历数组，跳过重复元素。代码如下：

```java
class Solution {
    public int findPairs(int[] nums, int k) {
        int cnt = 0;
        Arrays.sort(nums);
        for(int i = 0, j = 0; i < nums.length; ++i) {
            for(j = Math.max(j, i+1); j < nums.length && nums[j] - nums[i] < k; ++j) ;
            if(j < nums.length && nums[j] - nums[i] == k) ++cnt;
            while(i < nums.length - 1 && nums[i+1] == nums[i]) ++i;
        }
        return cnt;
    }
}
```

算法首先对数组进行排序。第一个for循环表示慢指针，遍历数组元素。嵌套的for循环中，快指针找到第一个元素，使得`nums[j]-nums[i] >= k`。if语句判断插值是否为k。嵌套的while循环跳过所有的重复值。

算法的时间复杂度为$O(n\log{n})$，空间复杂度为$O(1)$。

## 相关文章

- [Two Pointer I](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers/)
- [Two Pointer II](https://www.hytheory.com/algorithm/Leetcode-from-day-one-two-pointers-II/)