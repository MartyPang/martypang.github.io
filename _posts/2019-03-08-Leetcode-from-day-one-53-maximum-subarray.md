---
layout: post
title: '[LeetCode From Day One] - 53. Maximum subarray'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Divide and Conquer
last_modified_at: 2019-03-08T10:35:19-05:00
---

本篇博文介绍LeetCode简单难度[53题最大子数组](https://leetcode.com/problems/maximum-subarray/description/)的几种解法。最大子数组问题也是一个算法中的一个经典问题。首先给出题目描述。

> 给定一个整数数组，寻找和最大的连续子数组（至少包含一个元素）。
> Example:   
&nbsp; &nbsp; Input: [-2,1,-3,4,-1,2,1,-5,4]  
&nbsp; &nbsp; Output: 6  
&nbsp; &nbsp; Explanation: 子数组[4,-1,2,1]的和最大，为6

博主总结了该问题的三种解法，分别是：
- $O(n)$复杂度的一趟遍历算法
- 分治思想（Divide and Conquer）
- 动态规划（DP）- 更新

# 一趟遍历

一趟数组遍历比较tricky，其思想是，从左至右，每访问一个数，算法检查加上这个数会不会让总和变大。`cursum`记录当前为止的子数组的和，`maxsum`记录全局最大的子数组的和。若当前子数组和加上一个数`nums[i]`后比·`nums[i]`要小，则以`nums[i]`开始一个新的子数组。以题目中的Input为例。当访问`nums[1]`时，当前子数组为[-2]，其和加上`nums[1]`小于1，则开启一个新的子数组[1]，并将`cursum`设置为1，`maxsum`同样设置为1。

```java
class Solution {
    /**
     * single one pass solution
     * time complexity: O(n)
     * space complexity: O(1)
     */
    public int maxSubArray(int[] nums) {
        if(nums.length == 0) return 0;
        int cursum = 0;
        int maxsum = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; ++i) {
            cursum += nums[i];
            cursum = cursum < nums[i] ? nums[i] : cursum;
            maxsum = maxsum < cursum ? cursum : maxsum;
        }
        return maxsum;
    }
}
```

该算法的时间复杂度为$O(n)$， 空间复杂度为$O(1)$。

# 分治

分治的解法是题目最后提示的。
> If you figure out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.

如果你已经AC了$O(n)$的解法（一趟遍历），试着给出更为精巧的分治解法。分治法的基本思想就是把一个大问题分解为若干个子问题，子问题有着与原问题相同的形式，不同的是规模更小。当规模小到一定程度即可以直接求出解。

对于最大子数组和问题，我们可以将数组分为左右两部分分别递归求解。对于左右两个数组来说，解一定是以下三种情况之一：
- 全部都在左数组
- 全部都在右数组
- 跨越`mid`位置

对于前两种情况可以递归的来求解，最后一种情况需要考虑合并左右两边最大的数组。我们可以这么做，从`mid`往左右两边移动，分别记录左右两边的最大值。那么合并后的最大值就是左最大值加上右最大值加上中间值。问题最终的返回解就是左，右，合并三个最大值的较大者。下面贴出分治的代码。

```java
class Solution {
    /**
     * divide and conquer approach
     * time complexity: O(nlogn)
     */
    public int maxSubArray(int[] nums) {
        return subArraySum(nums, 0, nums.length-1);
    }

    private int subArraySum(int[] nums, int left, int right) {
        if(left > right) return 0;
        if(left == right) return nums[left];
        int mid = left + (right - left) / 2;
        int lmax = subArraySum(nums, left, mid);
        int rmax = subArraySum(nums, mid + 1, right);

        int maxleft = Integer.MIN_VALUE, maxright = Integer.MIN_VALUE;
        for(int i = mid, sum = 0; i >= left; --i) {
            sum += nums[i];
            maxleft = maxleft < sum ? sum : maxleft;
        }
        for(int i = mid + 1, sum = 0; i <= right; ++i) {
            sum += nums[i];
            maxright = maxright < sum ? sum : maxright;
        }
        return Math.max(Math.max(lmax, rmax), maxleft+maxright);
    }
}
```

分治的时间复杂度为$O(n\log n)$。

# 动态规划 - 更新

本身没怎么接触动态规划这种思路的，在LeetCode题目看discussion的时候发现不少人用DP来解决这个问题，在此学习记录一下。后续的文章应该会专门学习以下动态规划（Dynamic Programming），这里简单介绍以下动态规划的解题思路。动态规划的第一步同样是将问题分为几个子问题，但是与分治不同的是，动态规划主要应用于解决子问题重叠的情况。对于一些重复利用的结果，动态规划只需计算一次并保存结果。之后确定问题的状态转移方程，即一个递推公式。对于最大子数组和问题，我们有如下的状态转移方程：
$$
dp[i] = \max(dp[i-1]+nums[i], nums[i])
$$
其中`dp[i]`表示以`nums[i]`结尾的最大子数组和的情形。我们求出了`dp`，其元素最大值就是题解。

```java
class Solution {
    /**
     * dynamic programming
     */
    public int maxSubArray(int[] nums) {
        if(nums.length == 0) return 0;
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        for(int i = 1; i < nums.length; ++i) {
            dp[i] = Math.max(dp[i-1]+nums[i], nums[i]);
        }
        int max = dp[0];
        for(int i = 1; i < dp.length; ++i) {
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```

可以看到上面的代码仍旧可以优化，最后求全局的最大值可以在第一个`for loop`中做掉，并且`dp`这个数组也可以由一个全局的最大值代替，空间上也可以节省。（这么改了以后，发现就是第一种做法啊~w(ﾟДﾟ)w

