---
layout: post
title: '[LeetCode From Day One] - 669. Trim a binary search tree'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Tree
last_modified_at: 2019-05-27T18:56:21-05:00
---

* 目录
{:toc}

本文继续分析LeetCode有关树的题目，修剪一个二叉搜索树，使得所有节点值落在闭区间`[L,R]`中。

> 给定一颗BST和上下界`L`与`R`，删去所有值不在此区间的节点，返回新的树。
> Example: 给定如下BST和区间`[1,3]`:   
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3   
&nbsp; &nbsp; &nbsp; &nbsp; /&nbsp;  \     
&nbsp; &nbsp; &nbsp; 0 &nbsp; &nbsp; &nbsp; 4    
&nbsp; &nbsp; / &nbsp; \    
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 2  
&nbsp; &nbsp; &nbsp; &nbsp; /
&nbsp; &nbsp; &nbsp; 1
Output: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3   
&nbsp; &nbsp; &nbsp; &nbsp; /    
&nbsp; &nbsp; &nbsp; 2     
&nbsp; &nbsp; /    
&nbsp; 1  


关于树的问题通常都能用递归来解决，本题也不例外，后面再给出迭代的解法。

## 递归

递归的思想很简单，由于该树为二叉查找树，即左节点的值小于根节点，右节点的值大于根节点。一个节点node的值根据区间`[L,R]`可以分为三种情况：
- node.val > R，说明最后的结果一定都落在node.left；
- node.val < L，说明结果一定在node.right右子树这边；
- L <= node.val <=R，则对node的左右子树分别递归调用trim；

代码也就很清晰了，就是按照上面三种情况写就完事了。

```java
class Solution {
    public TreeNode trimBST(TreeNode root, int L, int R) {
        if(root == null) return null;
        if(root.val > R) return trimBST(root.left, L, R);
        if(root.val < L) return trimBST(root.right, L, R);

        root.left = trimBST(root.left, L, R);
        root.right = trimBST(root.right, L, R);

        return root;
    }
}
```

## 迭代解法

迭代的解法的思想，首先我们要找到一个有效的合法的根节点，即root.val一定是在区间`[L,R]`之间的。找到合法的root之后，再分别去左右子树找最接近L和R的节点。

```java
class Solution {
    public TreeNode trimBST(TreeNode root, int L, int R) {
        if(root == null) return null;
        while(root != null && (root.val < L || root.val > R)) {
            if(root.val > R) root = root.left;
            if(root.val < L) root = root.right; 
            
        }
        TreeNode tmpL = root;
        while(tmpL != null) {
            while(tmpL.left != null && tmpL.left.val < L) {
                tmpL.left = tmpL.left.right;
            }
            tmpL = tmpL.left;
        }
        TreeNode tmpR = root;
        while(tmpR != null) {
            while(tmpR.right != null && tmpR.right.val > R) {
                tmpR.right = tmpR.right.left;
            }
            tmpR = tmpR.right;
        }
        return root;
    }
}
```

我们看代码，第一个while循环找到一个根节点，它的值在`[L,R]`之间。接下来我们分别对左右子树进行trim。以左子树为例，如果`tmpL`左孩子的值小于L，说明`tmpL`的左孩子包括其左孩子的左子树均不符合条件，则令`tmpL`的左孩子等于当前左孩子的右孩子。有点绕口，举个最简单的例子来说，假设给定区间为`[3,4]`，当前`tmpL`为节点4，我们发现节点4的左节点的值小于L，则将3变为节点4的左节点。

```
               4
              /
             2  
            / \
           1   3
```

对于右子树的处理同理。

## 相关文章

[树的基本问题](https://www.hytheory.com/algorithm/Leetcode-from-day-one-basic-data-structure-tree/)   
[树的子树](https://www.hytheory.com/algorithm/LeetCode-from-day-one-572-subtree-of-another-tree/)