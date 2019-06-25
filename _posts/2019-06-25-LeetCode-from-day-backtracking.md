---
layout: post
title: '[LeetCode From Day One] - Backtracking'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Backtracking
last_modified_at: 2019-06-25T06:31:21-05:00
---

回溯法作为五大常用算法（分治，动态规划，贪心，回溯，条件分支）之一，经常用来解决枚举问题或者优化问题。常规的做法是，增量地构建solution，如果不满足，则回退一步，尝试其他选择。举个简单的迷宫的例子。如下图，进入迷宫到达1后，我们面临两种选择，假设走了2，就从出口出去了。假设走了3，我们发现死路一条，于是回溯到1，重新选择。

![backtrack](/images/20190625/backtrackeg.png){:  .align-center}

回溯法是对所有解的解空间树应用DFS策略，从根节点出发不断向叶子节点前进。当添加上某一节点后不满足问题的解，则向上层回溯直到根节点。该过程可以用如下递归的伪代码表示：

```
void backtrack(n, other params) :
    if (found a solution) :
        recordSolution();
        return;

    for (val = first to last) :
        if (isValid(val, n)) :
            applyValue(val, n);
            backtrack(n+1, other params);
            removeValue(val, n);
```

可用回溯法解决的问题一般有以下三类：

- 子集问题
- 组合问题
- 排列问题

典型的8皇后问题这里就不在详细说了，网上资料一堆。

* 目录
{:toc}

# 子集问题

[子集问题](https://leetcode.com/problems/subsets/)要求生成给定数组的所有子集。这个问题解法挺多种，比如遍历+辅助List，位操作和回溯法。本题可以直接套用递归回溯算法的框架。我们用一个list来保存当前的子集。每次调用backtrack函数，首先将当前子集加入到最终结果`res`中，之后把区间`[start, nums.length]`内的数一个个加入当前子集。回溯后，移除list最后一个元素，遍历下一个元素。

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        backtrack(res, new ArrayList<>(), nums, 0);
        return res;
    }
    
    private void backtrack(List<List<Integer>> res, List<Integer> curList, int[] nums, int start) {
        res.add(new ArrayList<>(curList));
        for(int i = start; i < nums.length; ++i) {
            curList.add(nums[i]);
            backtrack(res, curList, nums, i+1);
            curList.remove(curList.size()-1);
        }
    }
}
```

## 含重复元素集合的所有子集

子集问题的一个变种，假设给定的数组中包含重复元素，如何生成所有子集？还是用回溯的方法，代码的关键在于如何去重，如果数组是有序的，那么去重就变得很简单，判断前后两个元素是否相同即可。进行回溯前，我们先对数组进行排序`Arrays.sort(nums)`，回溯过程中，去重即可。

```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        backtrack(res, new ArrayList<>(), nums, 0);
        return res;
    }
    
    private void backtrack(List<List<Integer>> res, List<Integer> curList, int[] nums, int start) {
        res.add(new ArrayList<>(curList));
        for(int i = start; i < nums.length; ++i) {
            if(i > start && nums[i] == nums[i-1]) continue;
            curList.add(nums[i]);
            backtrack(res, curList, nums, i + 1);
            curList.remove(curList.size()-1);
        }
    }
}
```

# 组合问题

[组合问题I](https://leetcode.com/problems/combination-sum/)严格意义上也可以认为是子集问题，加了限定条件的子集问题。可以这么理解，找到数组的所有元素和为`target`的子集。在上面子集问题的代码基础上，加以修改就可得到组合问题的解。值得注意的是，虽然在组合问题I中给定的数组不包含重复元素，但是每个元素可以使用无限次。要实现这一点，只需要在for循环的start做手脚。可以观察到，我们令下一次backtrack的起始位置仍旧为`i`，实现了元素的重复使用。

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        backtrack(res, new ArrayList<>(), target, candidates, 0);
        return res;
    }
    
    private void backtrack(List<List<Integer>> res, List<Integer> curList, int remain, int[] nums, int start) {
        if(remain < 0) return;
        if(remain == 0) res.add(new ArrayList<>(curList));
        else {
            for(int i = start; i < nums.length; ++i) {
                curList.add(nums[i]);
                //use nums[i] for multiple times
                backtrack(res, curList, remain-nums[i], nums, i);
                curList.remove(curList.size()-1);
            }
        }
    }
}
```

## 含重复元素数组的组合问题

[Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)要求在给定数组包含重复元素的前提下，给出不含重复元素的解。与第二种子集问题一致，本题同样需要在回溯过程中去重。

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(candidates);
        backtrack(res, new ArrayList<>(), target, candidates, 0);
        return res; 
    }
    
    private void backtrack(List<List<Integer>> res, List<Integer> curList, int remain, int[] nums, int start) {
        if(remain < 0) return;
        if(remain == 0) res.add(new ArrayList<>(curList));
        else {
            for(int i = start; i < nums.length; ++i) {
                //avoid duplicate
                if(i > start && nums[i] == nums[i-1]) continue;
                curList.add(nums[i]);
                //use i+1 for next start pos
                //for not allowing multiple uses
                backtrack(res, curList, remain-nums[i], nums, i+1);
                curList.remove(curList.size()-1);
            }
        }
    }
}
```

## 手机号码生成的字母组合

本题是LeetCode中等难度[17题](https://leetcode.com/problems/letter-combinations-of-a-phone-number/description/)，题目要求给出一个手机号能代表的所有字母组合。

> 给定一个由数字2-9组成的字符串，返回该数字能代表的所有字母组合。数字与字母的映射与手机九宫格一致。
> Example:  
&nbsp; &nbsp; &nbsp; &nbsp; Input: "23"  
&nbsp; &nbsp; &nbsp; &nbsp; Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"]  

![backtrack](/images/20190625/keypad.png){:  .align-center}

每次读取一个数字，选择该数字映射的一个字母往下搜索。读取完所有数字后，就得到一个valid字母组合。本题还有个点在于如何计算数字对应的字母。观察到只有数字`7`和数字`9`对应4个字母，其余数字都只对应3个字母。所以对8和9做特殊处理即可。由于涉及到对String的频繁操作，本solution用StringBuilder保存临时字母组合。

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        List<String> res = new ArrayList<>();
        if(digits == null || digits.length() == 0 || digits.indexOf("1") >= 0) return res;

        StringBuilder sb = new StringBuilder();
        backtrace(res, sb, 0, digits);
        return res;
    }

    private void backtrace(List<String> res, StringBuilder phone, int level, String digits) {
        // a valid phone
        if(level == digits.length()) {
            res.add(phone.toString());
        }
        else {
            int cnt = (digits.charAt(level) == '9' || digits.charAt(level) == '7') ? 4 : 3;
            int start;
            if(digits.charAt(level) == '8') {
                start = 19;
            } else if (digits.charAt(level) == '9') {
                start = 22;
            } else {
                start = (Integer.parseInt(digits.substring(level, level+1)) - 2) * 3;
            }
            for(int i = 0; i < cnt; i++) {
                char c = (char)('a' + start + i);
                phone.append(c);
                backtrace(res, phone, level+1, digits);
                phone.deleteCharAt(phone.length() - 1);
            }
        }
    }
}
```

# 排列问题

排列问题代码跟组合问题差不多，不一样的地方在于排列需要使用全部的数字。这里简单贴一下代码。

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        backtrack(res, new ArrayList<>(), nums);
        return res;
    }

    private void backtrack(List<List<Integer>> res, List<Integer> curList, int[] nums) {
        if(curList.size() == nums.length) {
            res.add(new ArrayList<>(curList));
        }
        else {
            for(int i = 0; i < nums.length; ++i) {
                if(curList.contains(nums[i])) continue;
                curList.add(nums[i]);
                backtrack(res, curList, nums);
                curList.remove(curList.size() - 1);
            }
        }
    }
}

```
