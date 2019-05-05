---
layout: post
title: '[LeetCode From Day One] - 501. Find mode in BST'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Inorder Traversal
last_modified_at: 2019-05-06T14:28:44-05:00
---

之前整理过一些关于[树的高度，遍历](https://www.hytheory.com/algorithm/Leetcode-from-day-one-basic-data-structure-tree/)等基本的算法。本文聊聊如何找到一颗BST中频数最高的值。

> 给定一颗BST，找到所有出现最多的值。
> Example:  
&nbsp; &nbsp; Input: [1,null,2,2]  
&nbsp; &nbsp; Output: 2

最直接的做法肯定是遍历树，用map或者其他结构记录每个元素出现的次数，输出最多的元素即可。但是题目有个follow up，能否在不适用额外存储的情况下完成。这就要求我们必须一边遍历一边完成最大频数的判断。我们使用中序遍历，`prev`保存之前的节点值，用来比较并计算频数。

```java
class Solution {
    Integer prev = null;
    int count = 1;
    int max = 0;
    public int[] findMode(TreeNode root) {
        if (root == null) return new int[0];
        
        List<Integer> list = new ArrayList<>();
        traverse(root, list);
        
        int[] res = new int[list.size()];
        for (int i = 0; i < list.size(); ++i) res[i] = list.get(i);
        return res;
    }
    
    private void traverse(TreeNode root, List<Integer> list) {
        if (root == null) return;
        traverse(root.left, list);
        if (prev != null) {
            if (root.val == prev)
                count++;
            else
                count = 1;
        }
        if (count > max) {
            max = count;
            list.clear();
            list.add(root.val);
        } else if (count == max) {
            list.add(root.val);
        }
        prev = root.val;
        traverse(root.right, list);
    }
}
```

由于事先不知道最大频数的元素有几个，没法给int数组分配大小，故采用list记录。若当前元素的count大于max，则清空链表，添加当前元素。

本题采用中序遍历的原因是在之前的文章中提到过BST的中序遍历是一个有序数组。即当前节点的val如果不等于`prev`，表明之后的节点均不可能等于`prev`，这里把count重新置为1。