---
layout: post
title: '[LeetCode From Day One] - Binary Search'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Binary Search
last_modified_at: 2019-04-24T14:40:38-05:00
---

二分查找算法是一种针对**有序数组**使用较为频繁的搜索算法。常规的遍历数组，与数组每个元素比较的做法时间复杂度为$O(n)$，而二分查找每次都能去除一半的查找区间，时间复杂度由$O(n)$降低到$O(\log n)$。今天做的LeetCode简单难度[35题查找插入位置](https://leetcode.com/problems/search-insert-position/description/)和[69题x的平方根](https://leetcode.com/problems/sqrtx/description/)均采用二分查找解决。

4.24更新LeetCode简单难度[278题找到第一个坏版本](https://leetcode.com/problems/first-bad-version/description/)。

二分查找的基本思想将有序数组中间位置的值与目标值比较，如果相等则返回中间位置；否则，如果中间位置的值大于目标值，则继续查找前半部分，如果小于，则查找后半部分。重复直到找到或者low小于high。 二分查找的实现方式有递归和循环两种，一般不写成递归，效率不高且存在栈溢出的可能。这里还是贴一下递归的代码。

```java
public int binarySearch(int[] array, int low, int high, int target) 
{
    if(low <= high) {
        int mid = (low + high) / 2;
        if(array[mid] == target) {
            return mid;
        }
        else if(array[mid] > target) {
            return binarySearch(array, low, mid - 1, target);
        }
        else {
            return binarySearch(array, mid + 1, high, target); 
        }
    }
    else {
        return -1;
    }
}
```

## 查找插入位置

> 给定一个有序数组和一个目标值，返回该目标值在数组中出现的下标。若不存在该目标值，则返回插入的下标。假定数组中没有重复值。
> Example:
&nbsp; &nbsp; Input: [1,3,5,6], 5
&nbsp; &nbsp; Output: 2

直接应用二分查找就完事了，若找到则返回`m`值。否则，返回插入下标，也就是跳出循环后的`low`。注意在计算中间位置`m`的时候，有如下两种计算方式：

- `m=(low+high)/2`
- `m=low+(high-low)/2`

若使用`(low+high)/2`，可能会溢出。将加法转化为减法，使用`low+(high-low)/2`。

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int low = 0;
        int high = nums.length-1;
        if(high < 0) return 0;
        while(low <= high) {
            int m = low + (high - low) / 2;
            if(nums[m] == target) {
                return m;
            }
            else if(nums[m] > target) {
                high = m-1;
            }
            else {
                low = m+1;
            }
        }
        return low;
    }
}
```

还有一个要注意的地方就是循环跳出条件与`high`的更新。当循环条件为`low <= high`时，`high`更新为m-1。当去除条件的等号，`high = m`。代码中的写法，跳出循环后，low比high大1，刚好是要插入的位置的下标。

## x的平方根

> 计算并返回一个非负整数x的平方根。
> Example: 
&nbsp; &nbsp; Input: 4
&nbsp; &nbsp; Output: 2
> Note: 8的平方根是2.82842...，函数返回2。

x的平方根一定在区间$[0, x]$之间，可以利用二分查找在区间内查找平方根。

```java
class Solution {
    public int mySqrt(int x) {
        if(x <= 1) return x;
        int low = 1;
        int high = x;
        while(low <= high) {
            int m = low + (high - low) / 2;
            int sqrt = x / m;
            if(sqrt == m) {
                return m;
            }
            else if(sqrt < m) {
                high = m - 1;
            }
            else {
                low = m + 1;
            }
        }
        return high;
    }
}
```

与上题不同的是，由于本题需要返回平方根的整数部分，最后的返回值不是`m`，就是`high`和`low`中的较小值。因为循环跳出条件为`low <= high`，故跳出循环后的`high`一定小于`low`，最终返回`high`。

## First Bad Version

题目抽象出来就是对于一个包含n个元素的数组`[1,2,3,...,n]`，找到第一个元素使得接口`isBadVersion(x)`返回true的元素x。

> 例如给定n=5，version=4是第一个坏版本。
> &nbsp; &nbsp; 调用`isBadVersion(3)`返回false；
> &nbsp; &nbsp; 调用`isBadVersion(5)`true；
> &nbsp; &nbsp; 调用`isBadVersion(4)`true；
> &nbsp; &nbsp; 最终找到4。

很直接的二分查找的应用，主要循环跳出条件，当`right`就比`left`大1的时候即跳出循环。此时只需判断`left`是否为bad version即可，若`left`不是，那么`right`一定是bad version。

```java
public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
        int left = 1;
        int right = n;
        while(right - left > 1) {
            int mid = left + (right-left) / 2;
            if(isBadVersion(mid)) right = mid;
            else left = mid + 1;
        }
        if(isBadVersion(left)) return left;
        else return right;
    }
}
```