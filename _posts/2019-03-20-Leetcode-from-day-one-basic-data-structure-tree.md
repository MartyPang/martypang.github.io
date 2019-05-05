---
layout: post
title: '[LeetCode From Day One] - Several Kinds of Tree Problem'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Tree
last_modified_at: 2019-03-20T14:29:27-05:00
---
树是一种基本的常见数据结构。与链表，队列，栈等线性结构不同，树的存储方式不再是线性。树由n个节点组成，每个节点有若干个孩子。

- 若`n==0`，则为一颗空树；
- 若`n==1`,则只包含一个根节点；
- 其他情况为正常的树；

数据结构中有很多的树结构，包括二叉树（Binary Tree）、二叉搜索树（Binary Search Tree，BST）、AVL树、红黑树、Trie树等等。后面会更新文章专门介绍各种树结构以及对应的操作。本文涉及的题目基本使用最基础的二叉树结构（毕竟easy难度<(ˉ^ˉ)>

```java
    //Definition for a binary tree node.
    public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;
        TreeNode(int x) { val = x; }
    }
```

关于树的最基本的问题基本为递归与遍历两类，更复杂的问题要么是对递归方法与遍历方法的应用，要么是结合特定类型的树的特性操作。一颗二叉树每个节点有两个指针，每个指针又指向一个树，是一种典型的递归结构，很多问题可以使用递归来解决。对树的遍历又有层次遍历与前中后序遍历。下面结合最近刷的题一一介绍。

# 递归

## 树的最大高度

先看个最简单的题，[104题求树的最大高度](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)。树的高度表示从根节点到叶子节点最远的距离，每加一层，高度加1。

```
 ⁠   3
 ⁠  / \
 ⁠ 9  20
 ⁠   /  \
 ⁠  15   7
```

例如上面这颗树的高度为3。看了图代码就出来了，递归，每次选择最大高度的子树加1，两行代码搞定。

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null) return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```

## 树的最小高度

上一题的变种，[111题求最小高度](https://leetcode.com/problems/minimum-depth-of-binary-tree/description/)。需要注意的是，简单地认为把上题解法的`max`函数换成`min`即可解决问题的想法是错误的。求最大高度时，总归能走到叶子节点，但是如果用同样的方式求最小高度，最后可能走到null，而非叶子节点。举个简单的例子：

```
 ⁠   3
 ⁠  / 
 ⁠ 9 
```
如果采用最大高度的做法，最终返回的最小高度的路径是`3-->null`，也就是1，但其中最小高度应该为2。所以在递归过程中，我们要判断叶子节点的情况。

```java
class Solution {
    public int minDepth(TreeNode root) {
        if(root == null) return 0;
        if(root.left == null && root.right == null) return 1;
        if(root.left == null) return minDepth(root.right) + 1;
        if(root.right == null) return minDepth(root.left) + 1;

        return Math.min(minDepth(root.left), minDepth(root.right)) + 1;
    }
}
```

第二个if表示当前的`root`为叶子节点，因为它没有左右孩子节点。第三个if表明叶子节点在`root`的右子树，同理第四个if表示叶子节点在左子树。最后一个return表明最短路径的叶子节点在哪边不清楚，继续递归，选高度小的。

## 判断一棵树是否是平衡树

简单难度110题[如何判断一棵树是平衡树](https://leetcode.com/problems/balanced-binary-tree/description/)。平衡树是指左右子树高度差都小于等于1的树。本题可利用104的solution，根据平衡树的定义，递归比较左右子树的高度差。

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root == null) return true;

        int ldepth = maxDepth(root.left);
        int rdepth = maxDepth(root.right);

        return Math.abs(ldepth-rdepth) <= 1 && isBalanced(root.left) && isBalanced(root.right);
    }

    public int maxDepth(TreeNode root) {
        if(root == null) return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```


# 遍历

树的各种遍历方式经常用到的数据结构有队列和栈，必须熟练掌握的算法深度优先搜索（DFS）与广度优先搜索（BFS）。

## 层次遍历

使用BFS进行树的层次遍历，用队列结构存储每层节点。最基本的两道题：[102题二叉树层次遍历 I](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)和[107题二叉树层次遍历 II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/description/)。

> 给定一颗二叉树，返回其层次遍历。（i.e., 从左往右，level by level）  
> Example：  
&nbsp; &nbsp; Input：[3,9,20,null,null,15,7]，  
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3  
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; / &nbsp; \   
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 9 &nbsp; &nbsp; 20  
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; / &nbsp; &nbsp; \    
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 15 &nbsp; &nbsp; &nbsp; &nbsp; 7  
&nbsp; &nbsp; Output：[ [3], [9, 20], [15, 7] ]  


```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        LinkedList<List<Integer>> result = new LinkedList<>();
        if(root == null) return result;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();
            for(int i = 0; i < size; ++i) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            result.addLast(level);
        }
        return result;
    }
}
```
107题只需把每层的list插入双向链表`result`的头部即可。

## 前中后序遍历

树的前中后序遍历也是最基本要掌握的知识。笔试题经常会碰到，给你中前序求后序或者给你中后序求前序。前中后序用递归来写是最直白的，但为了与上题有所区分，本小节均用非递归的循环写法。

### 前序遍历

前序遍历/先序遍历的访问顺序是根左右。

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList();
        if(root == null) return result;
        Stack<TreeNode> stack = new Stack();
        stack.push(root);
        while(!stack.isEmpty()) {
            TreeNode top = stack.pop();
            result.add(top.val);
            if(top.right != null) stack.push(top.right);
            if(top.left != null) stack.push(top.left);
        }
        return result;
    }
}
```

注意栈是先进后出的。所以`right`先进栈，`left`后入栈确保`left`先于`right`被访问。

### 后序遍历

后序遍历的顺序是左右根，它可以由前序遍历调换左右遍历的反序得到，即`root-->right-->left`的反序结果。如果使用java的话可以采用双向链表数据结构，$O(1)$的复杂度添加到链表最前。

```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        LinkedList<Integer> result = new LinkedList<>();
        if(root == null) return result;
        Stack<TreeNode> stack = new Stack();
        stack.push(root);
        while(!stack.isEmpty()) {
            TreeNode top = stack.pop();
            result.addFirst(top.val);
            if(top.left != null) stack.push(top.left);
            if(top.right != null) stack.push(top.right);
        }
        return result;
    }
}
```

### 中序遍历

中序遍历的非递归写法，首先我们一直选左子树走直到走到头，之后再pop一个访问一个，最后再访问所有的右子树。一颗二叉搜索树（BST）的中序遍历是一个有序数组，左节点小于根节点，右节点大于根节点。

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode tmp = root;
        while(tmp != null || !stack.isEmpty()) {
            if(tmp != null) {
                stack.push(tmp);
                tmp = tmp.left;
            }
            else {
                TreeNode node = stack.pop();
                result.add(node.val);
                tmp = node.right;
            }
        }
        return result;
    }
}
```

### 由前/后序与中序推导后/前序

最后回到我们的笔试/面试题，给你中序以及前序求后序，或则中序以及后序求前序。如果是选择题，那画树求解是比较方便的一种做法。这里以中等难度105题[以前序和中序构造二叉树](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)为例介绍如何画树求解。

> 给定一颗树的前序遍历与中序遍历结果，试构造相应的二叉树。
> Example:  
&nbsp; &nbsp; Input: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]  

首先，根据前序遍历的特点可知节点3是树的根节点。第二步根据根节点将中序分成两半`[9]`和`[15,20,7]`，分别是根节点3的左右子树。之后观察左子树，左子树的根节点是节点3的左孩子，一定出现在前序遍历节点3的后面，即节点9是节点3的左孩子。再观察右子树，在前序遍历中，一定是先把根节点以及根节点的所有左子树节点遍历完之后才会遍历右子树。20是第一个出现在前序遍历的节点，所以右子树的根节点也就是节点3的右孩子是节点20。以上过程是递归的，再分别对左右子树的根节点重复上述过程直到遍历完。最终的结果是：

```
 ⁠   3
 ⁠  / \
 ⁠ 9  20
 ⁠   /  \
 ⁠  15   7
```

该树的后序遍历为`[9,15,7,20,9]`。

算法思想与画树的过程类似，要注意数组的上下界。算法中`preleft`指向的元素是当前的根节点，递归地生成其左右孩子节点再返回。每次递归，将preorder与inorder切成子序列，子序列的根就是左右孩子节点。

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if(preorder.length == 0 || inorder.length == 0) return null;
        return constructNode(preorder, 0, preorder.length-1, inorder, 0, inorder.length-1);
    }

    public TreeNode constructNode(int[] preorder, int preleft, int preright, int[] inorder, int inleft, int inright) {
        if(preleft > preright || inleft > inright) return null;
        TreeNode node = new TreeNode(preorder[preleft]);
        int rootindex = 0;
        for(; rootindex < inright; ++rootindex) {
            if(inorder[rootindex] == preorder[preleft]) break;
        }
        node.left = constructNode(preorder, preleft+1, preleft+(rootindex-inleft), inorder, inleft, rootindex-1);
        node.right = constructNode(preorder, preleft+(rootindex-inleft)+1, preright, inorder, rootindex+1, inright);
        return node;
    } 
}
```

中等难度106题[以后序和中序构造二叉树](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/description/)的解法与上题类似。无非是preorder从左往右遍历，postorder则从右往左。

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        if(postorder.length == 0 || inorder.length == 0) return null;
        return constructNode(postorder, postorder.length-1, inorder, 0, inorder.length-1);
    }

    public TreeNode constructNode(int[] postorder, int postid, int[] inorder, int inleft, int inright) {
        if(postid < 0 || inleft > inright) return null;

        TreeNode node = new TreeNode(postorder[postid]);
        int rootindex = 0;
        for(; rootindex <= inright; ++rootindex) {
            if(inorder[rootindex] == postorder[postid]) break;
        }
        node.left = constructNode(postorder, postid-(inright-rootindex)-1, inorder, inleft, rootindex-1);
        node.right = constructNode(postorder, postid-1, inorder, rootindex+1, inright);
        return node;
    }
}
```