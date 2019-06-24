---
layout: post
title: '[LeetCode From Day One] - Backtracking'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Backtracking
last_modified_at: 2019-06-25T16:31:21-05:00
---



回溯法作为五大常用算法（分治，动态规划，贪心，回溯，条件分支）之一，经常用来解决枚举问题或者优化问题。常规的做法是，增量地构建solution，如果不满足，则回退一步，尝试其他选择。举个简单的迷宫的例子。如下图，进入迷宫到达1后，我们面临两种选择，假设走了2，就从出口出去了。假设走了3，我们发现死路一条，于是回溯到1，重新选择。

回溯法是对所有解的解空间树应用DFS策略，从根节点出发不断向叶子节点前进。当添加上某一节点后不满足问题的解，则向上层回溯。该过程可以用如下递归的伪代码表示：

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

# 子集问题

子集问题要求生成给定数组的所有子集。这个问题解法挺多种，比如遍历+辅助List，位操作和回溯法。
