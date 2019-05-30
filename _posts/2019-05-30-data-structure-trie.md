---
layout: post
title: 'Understand Trie'
author: Marty Pang
image:
  path: /images/20190530/cover.jpg
  thumbnail: /images/20190530/cover.jpg
categories: 
  - Data Structure
tags: 
  - Trie
  - Tree
last_modified_at: 2019-05-30T09:41:28-05:00
---

几乎所有的搜索树都是用来存储数值的集合。而Trie树（发音为try）又称为字典树，基数树（radix tree）或者前缀树（prefix tree），是一种特殊的搜索树，经常用来存储和搜索字符串。有几种用来表示一组单词的数据结构例如哈希表，平衡二叉搜索树（红黑树，AVL树等）。但这些方式都是以整个单词为一个key进行存储，空间上比较节省，但是搜索效率不高，比如平衡树的搜索时间复杂度为$O(m\log{n})$，其中m为单词的最大长度，n为所有的单词数。并且如果使用哈希的话，存在碰撞的情况。而Trie的提出，让字符串的搜索时间可以优化到$O(m)$，其中m为key的最大长度。由于Trie中每个节点只表示一个字母，所以会带来更多的存储开销，这是一种典型的空间换时间策略。本文介绍Trie的基本结构以及常用操作包括插入，查询和删除。


* 目录
{:toc}

# 基本结构

Trie除根节点以外的节点包含一个字母和若干个孩子节点。每条从根节点出发到任意节点的路径都可能表示一个单词。

![Trie node](/images/20190530/trie.png){:	.align-center}

以下Java代码表示了一个Trie节点的基本结构。根节点的value为空，其他节点的value表示一个字符。如果在只有英文字符的上下文中使用，可把`CHAR_SIZE`改为26，一个节点最多有26个孩子节点。一组孩子节点可以用数组或者链表存储，这里采用HashMap是为了节省空间。当然由于26个字母的顺序是唯一确定的，也可以开辟一个大小为26的TrieNode数组，按下标进行访问。但是大部分节点都不会有满满当当26个孩子，用数组表示会存在大量的空指针，浪费空间。为了区分普通节点与一个单词最后一个节点，我们用一个标志位`isEndOfWord`表示该节点是单词的最后一个字母。

```java
public class TrieNode {
    static final int CHAR_SIZE = 256;

    public String value;
    //Array: TrieNode[] children;
    public HashMap<Character, TrieNode> children;
    public boolean isEndOfWord;

    public TrieNode() {
        children = new HashMap<>(CHAR_SIZE);
        isEndOfWord = false;
    }
}
```

# 操作

对Trie树的操作包括插入一个单词，查找一个单词是否存在以及删除一个单词。Trie树本质上就是一颗树，所以我们用递归形式来定义Trie，TrieTree类只包含一个root节点。

```java
public class TrieTree {
    private TrieNode root;

    public TrieTree() {
        root = new TrieNode();
    }
    //function definition
    //...
}
```

## 插入

往Trie插入一个key的过程很简单。`word`的每个字母都作为一个独立的节点，如果该节点在孩子节点中不存在，则需要new一个，插入children。对于`word`的最后一个节点，设置`isEndOfWord`为`true`。如果待插入的`word`是已有单词的一个prefix，同样设置最后一个节点的`isEndOfWord`属性为`true`。插入单词的时间复杂度为$O(n)$，n为`word`的长度。

```java
/**
 * insert a word to trie
 * Time complexity: O(n)
 * @param word the inserted word
 */
public void insertWord(String word) {
    if(word.isEmpty()) return;
    TrieNode cur = root;
    for(char c : word.toCharArray()) {
        TrieNode next = cur.children.get(c);
        if(next == null) {
            next = new TrieNode();
            next.value = c+"";
            cur.children.put(c, next);
        }
        cur = next;
    }
    cur.isEndOfWord = true;
}	
```

## 查询

查询一个`word`是否存在的过程与insert类似。同样自顶向下往下找匹配的子节点，与insert不同的是，一旦没有找到一个匹配的节点，方法直接返回false。若果找到表示最后一个字母的节点，判断其`isEndOfWrod`属性是否为真即可。查询的时间复杂度同样为$O(n)$。

```java
/**
 * if word exists in trie
 * @param word
 * @return true if exists
 */
public boolean search(String word) {
    TrieNode cur = root;
    for(char c : word.toCharArray()) {
        if(!cur.children.containsKey(c+"")) return false;
        cur = cur.children.get(c+"");
    }
    return cur.isEndOfWord;
}
```

## 删除

删除操作要比前两个来的复杂一点。我们先来分析待删除的单词在Trie中可能出现的情况，有如下四种：
- 单词在Trie中不存在，这种情况delete操作不能修改Trie树；
- 表示单词的路径上不包含其他单词，即是一个独立的分支，删除路径的所有节点即可；
- 该单词映射的路径是其他单词的一个前缀，这种情况只需要把该单词的最后一个节点的`isEndOfWord`设置为`false`；
- 该单词的前缀中还存在至少一个单词，删除另一个单词以后的所有节点即可；

不同于前两个自顶向下的操作，删除节点时，如果先删了父节点，就找不到子结点了，故我们采用自底向上的方式删除节点，算法用递归实现。

```java
public void delete(String word) {
    delete(root, word, 0);
}

public boolean delete(TrieNode node, String word, int index) {
    if(index == word.length()) {
        // word does not exist in the trie
        if(!node.isEndOfWord) {
            return false;
        }
        // if node has no other child, it's safe to delete word
        node.isEndOfWord = false;
        return node.children.isEmpty();
    }
    char c = word.charAt(index);
    TrieNode next = node.children.get(c);
    if(next == null) {
        return false;
    }
    boolean shouldDelete = delete(next, word, index+1) && !node.isEndOfWord;

    if(shouldDelete) {
        node.children.remove(c);
        // make sure there is only one word to be deleted
        return node.children.isEmpty();
    }
    return false;
}
```

# Trie的应用

Trie树的应用包括但不限于字符串的匹配，字典序排序（Trie树的前序遍历结果），还有自动补全功能，类似谷歌搜索时的功能。

![auto-complete](/images/20190530/google.gif){:	.align-center}

## 字典中的最长单词

本题是LeetCode简单难度720题[找到字典中的最长单词](https://leetcode.com/problems/longest-word-in-dictionary/)。

> 给定一个单词数组表示英文字典，找到一个最长的单词，它可以由其他单词一次加一个字母构成。如果答案不止一个，则返回字典序最小的一个。  
> Example:  
&nbsp; &nbsp; &nbsp; &nbsp; Input: `words=["a", "banana", "app", "appl", "ap", "apply", "apple"]`  
&nbsp; &nbsp; &nbsp; &nbsp; Output: `"apple"`  
"apply"和"apple"均符合最长单词的要求，"apple"字典序更小，故返回。

知道Trie树结构的话，题目很明显在提示使用Trie树。算法思想就是遍历`words`，构建一个Trie树，然后使用DFS遍历Trie树，找到一条最长路径，使得路径上每个节点都是字典里的一个单词的最后一个字母。与上面Trie树实现略有出入的是，我们用一个整型变量代替布尔变量，既能表示节点是否是单词结尾，又表示该单词在`words`数组中的位置，方便获取。

在DFS的过程中，我们只把root节点或者是单词结尾的节点的孩子加入到栈中，因为题目要求最长单词必须由其他单词不断加一个字母构成。

```java
class Solution {
    class TrieNode {
        public String value;
        public Map<Character, TrieNode> children = new HashMap<>();
        //boolean isEndOfWord = false;
        int endIndex = 0;
    }

    class Trie {
        public TrieNode root = new TrieNode();
        
        public void insert(String word, int index) {
            TrieNode cur = root;
            for(char c : word.toCharArray()) {
                TrieNode next = cur.children.get(c);
                if(next == null) {
                    next = new TrieNode();
                    next.value = c+"";
                    cur.children.put(c, next);
                }
                cur = next;
            }
            cur.endIndex = index;
        }
    }
    public String longestWord(String[] words) {
        Trie tree = new Trie();
        tree.root.endIndex = 0;
        for(int i = 0; i < words.length; ++i) {
            tree.insert(words[i], i+1);
        }
        String res = "";
        //dfs
        Stack<TrieNode> stack = new Stack<>();
        stack.push(tree.root);
        while(!stack.isEmpty()) {
            TrieNode node = stack.pop();
            // check if node represents a word from words
            if(node.endIndex > 0 || node == tree.root) {
                if(node != tree.root) {
                    String word = words[node.endIndex - 1];
                    // longest length or smallest lexicographical order
                    if(word.length() > res.length() || word.length() == res.length() && word.compareTo(res) < 0) {
                        res = word;
                    }
                }
                for(TrieNode n : node.children.values()) {
                    stack.push(n);
                }
            }
        }
        return res;
    }
}
```