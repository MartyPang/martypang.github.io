---
layout: post
title: '[LeetCode From Day One] - 572. Subtree of Another tree'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Tree
last_modified_at: 2019-05-09T16:46:20-05:00
---

本篇文章分析LeetCode简单难度572题，判断一棵树是否为另一棵树的子树。这题也是包括facebook，google在内的大公司考察的关于树的经典题目。

> 给定两颗非空的二叉树`s`与`t`，写一个函数判断`t`是否与`s`的某颗子树有着相同的结构。
> Example: given tree s and t:   
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4  
&nbsp; &nbsp; &nbsp; &nbsp; /&nbsp;  \  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; / &nbsp; \   
&nbsp; &nbsp; &nbsp; 4 &nbsp; &nbsp; &nbsp; 5  &nbsp; &nbsp;  1 &nbsp; &nbsp; 2  
&nbsp; &nbsp; / &nbsp; \   &nbsp; &nbsp; &nbsp;  
&nbsp; 1 &nbsp; &nbsp; 2  &nbsp; &nbsp; &nbsp;  
Output: true

`s`的子树由`s`中的任意一个节点以及该节点的所有子孙节点组成。`s`本身也是`s`的子树。

![subtree](/images/20190510/subtree.png){:	.align-center}

本文提供两种思路，分别是遍历`s`，判断以遍历节点为根节点的子树与`t`是否相同和前序遍历`s`与`t`，判断`t`的遍历结果是否为`s`的字串。


## 遍历`s`

先看第一种方式，比较直观。首先我们递归地在`s`中找到与`t`根节点值相同的节点（`if(s.val == t.val`）。之后再递归地（`isEqual`）判断以该节点为根节点构成的子树与`t`是否相同。递归函数`isEqual`中，第一个if判断表示对应的节点均为`null`，返回true；若走到第二个条件，`s`与`t`当中有一个为`null`，返回false；最后再判断两个节点的值。

```java
class Solution {
    public boolean isSubtree(TreeNode s, TreeNode t) {
        if(s == null) return false;
        return isEqual(s, t) || isSubtree(s.left, t) || isSubtree(s.right, t);
    }

    private boolean isEqual(TreeNode s, TreeNode t) {
        if(s == null && t == null) return true;
        if(s != null && t == null || s== null && t != null) return false; 
        if(s.val == t.val) return isEqual(s.left, t.left) && isEqual(s.right, t.right);
        return false;
    }
}
```

在最坏情况下，对于`s`中的每个节点，我们都需要判断相应的子树与`t`是都相同，算法时间复杂度为$O(n\times m)​$，其中n为`s`的节点数，m为`t`的节点数。空间复杂度即递归栈的深度为$O(n)$。

## 前序遍历`s`与`t`

第二种方式分别得到`s`与`t`的前序遍历字符串，判断`t`是否为`s`的字串即可。与一般的前序遍历不同的是，本题需要注意左右孩子为空的情况，需要使用特殊字符来标记空的左孩子与空的右孩子。举个例子说明，假设我们有如下两棵树：

```
     1         1
    /           \
   1             1
```

可以得到`s`的前序遍历结果为`11`，`t`的前序遍历也为`11`，但是`t`却不是`s`的子树。如果我们用"lnull"与"rnull"分别标记空的左孩子与空的右孩子，则`s`的前序遍历变为`11rnull`，`t`的为`1lnull1`，此时后者的结果不是前者的字串。

```java
class Solution {
    public boolean isSubtree(TreeNode s, TreeNode t) {
        String pres = preorder(s, true);
        String pret = preorder(t, true);

        return pres.indexOf(pret) >= 0;
    }

    private String preorder(TreeNode n, boolean isLeft) {
        if(n == null) {
            if(isLeft) return "lnull";
            else return "rnull";
        }
        return "#" + n.val + preorder(n.left, true) + preorder(n.right, false);
    }
}
```
后序遍历也可以做，但是中序遍历不行。另外空左节点与空右节点可以统一用"null"表示，不作区分也可。
分别遍历`s`与`t`时间复杂度为$O(n+m)$，`indexOf`复杂度为$O(n \times n)$，所以总的时间复杂度为$O(n+m+n \times m)$。当然判断子串也可以采用KMP这种复杂度为$O(m+n)$的方法。空间复杂度为两个递归栈深度的和$O(n+m)$。可以看到这种方式更加耗费内存，对于节点数多的树不太适合。

