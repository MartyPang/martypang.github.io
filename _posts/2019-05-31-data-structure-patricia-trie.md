---
layout: post
title: 'Compressing Radix tree - Patricia Trie'
author: Marty Pang
categories: 
  - Data Structure
tags:
  - Patricia
  - Trie
  - Tree
last_modified_at: 2019-05-31T16:34:07-05:00
---

之前的[文章](https://www.hytheory.com/data%20structure/data-structure-trie/)介绍了Trie树的基本结构与实现。Trie利用单词之间共享前缀的特点，能够高效的进行查询，查询效率只与key的长度有关。但是Trie树存在的主要问题是，由于每个节点只存储一个字母，整棵树会很稀疏，导致存储开销很高。再加上，如果每个节点采用数组存储孩子节点，将会存在大量的空指针。[
Donald R. Morrison](https://dl.acm.org/author_page.cfm?id=81547818856&coll=DL&dl=ACM&trk=0)等人最早在[Practical Algorithm To Retrieve Information Coded in Alphanumeric](<https://dl.acm.org/citation.cfm?id=321481>)论文中提出了Patricia Trie结构，它是一颗空间优化的Trie树，旨在压缩Trie的内存使用。Patricia的一个基本思想就是只有一个孩子的节点与它的父节点合并成一个节点。下图是一个演示Patricia压缩Trie树的简单例子。

|      |      |
| :--: | :--: |
|   ![Trie](/images/20190531/trie.png)  |    ![Patricia Trie](/images/20190531/patricia.png)   |

左图表示用常规的Trie树存储`apple`和`apply`两个单词，可以看到树高很高，并且节点数也很多。而右图的Patricia Trie把两个单词的公共前缀`appl`用一个节点表示从而压缩了存储。换句话说，Patricia的节点只存储正真的分支，对于只包含一个孩子的节点采用如上图所示的压缩表示。

本文介绍Patrcia Trie的基本结构以及常用操作包括插入，查询和删除。


* 目录
{:toc}

# 基本结构

对于常规的Trie，其节点可能定义如下：

```java
public class TrieNode {
    static final int ALPHABET_SIZE = 26;

    public TrieNode[] children;
    public boolean isEndOfWord;
}
```

每个节点中都会有一个数组存储该节点的孩子节点的引用，并且孩子节点是按字典序排列的，例如children[0]表示'a'。然而对于Patricia Trie的节点来说，假设叶子节点存储的是<key,value>键值对，结构如下，其中key存储的可能是部分键，也能对应一个完整的键。

```java
public class PatriciaTrieNode {
    public String key;
    public Integer value;
    List<PatriciaTrieNode> children;
    boolean isTerminal;
}
```

正如前面的例子演示的，与Trie树用char标记边的做法不同，Patricia Trie树中的边用一个字符串标记。children用LinkedList实现，孩子节点在链表中按字典序排序。

# 操作

与Trie树相同，Patricia Trie同样支持搜索，插入，删除操作。首先我们定义一颗Patricia Trie树。

```java
public class PatriciaTrie {
    PatriciaTrieNode root;

    public PatriciaTrie() {
        root = new PatriciaTrieNode();
    }
}
```

## 搜索

对Patricia Trie的插入与删除操作会涉及到节点的split与merge，我们先介绍不复杂的search操作。

```java
public int getEqualIndex(String searchKey) {
    int len = Math.min(searchKey.length(), key.length());
    int j = 0;
    for(; j < len; ++j) {
        if(searchKey.charAt(j) != key.charAt(j)) break;
    }
    return j;
}
```

一开始我们在PatriciaTrieNode类中定义一个函数`getEqualIndex`，返回两个key相同部分的长度，或者说第一个不相同字符的下标。三个操作都会涉及到这个函数。

search的过程遍历当前节点的所有孩子节点。对于每一个child，首先得到第一个不同字符的小标index。如果index为0，说明key与child.key没有相同的前缀。判断条件`key.compareTo(child.key) < 0`返回true，表明该key与之后的child也不可能存在公共前缀。若index不为0，即`0<index<=min length`，我们又分为两种情况讨论：

**`0<index<min length`**

这种情况下，child.key与key只有部分重叠，不符合搜索条件，直接返回。举个例子，假设当前节点为`ab`，child为`efg#`，#表示终结符，key为`efb`，index为2，很明显不存在为`efb`的key。

```
         ab       
        /  \
       d    efg# 
```

**`index == min length`**

根据key和child.key长度的关系分为三种情况：
- key的长度与child.key的长度相等， 只需检查isTerminal属性是否为true即可；
- key的长度小于child.key的长度，也就是child.key包含更多的字符，停止搜索返回null；
- key的长度大于child.key的长度，如注释，key截取掉公共前缀，递归调用search函数；

```java
private Integer search(PatriciaTrieNode node, String key) {
    for(PatriciaTrieNode child : node.children) {
        int index = child.getEqualIndex(key);
        if(index == 0) {
            if(key.compareTo(child.key) < 0) return null;
            continue;
        }
        // key and node.key are partially overlapped
        // child.key="abc", key="abd", index=2
        if(index < Math.min(key.length(), child.key.length())) return null;
        // child.key="abc", key="abc"
        if(key.length() == child.key.length()) {
            // find a match
            if(child.isTerminal) return child.value;
            return null;
           }
        // child.key="abc", key="ab", index=2
        else if (key.length() < child.key.length()) {
            return null;
        }
        //child.key="ab", key="abc", remains="c"
        else {
            String remains = key.substring(index);
            return search(child, remains);
        }
    }
    return null;
}
```

## 插入

插入的过程与搜索类似，不同的是，我们需要考虑节点的分裂情况。在`index==0`的情况下，若key值字典序小于child.key，则在child之前插入一个新的孩子节点；若当前节点的最后一个节点的字典序仍小于key，则在当前节点的孩子链表尾部插入一个新节点。

在`index==min length`的情况下，在key长度小于child.key长度时，要考虑child节点的分裂。分裂节点继承当前child节点的所有孩子。而child节点则新生成一个孩子链表，插入分裂节点，并且将isTerminal置为true，因为插入了新的key。而当key长度大于child.key长度时，则递归地对剩余部分的key调用insert操作。

当`index<min length`时，则需要分裂child节点和key对应的新节点，并且按照字典序插入两个分裂节点。

最后跳出for循环的时候，要判断以下孩子节点的数量是否为0，例如对一个空树来说，直接插入即可。

```java
public void insert(PatriciaTrieNode node, String key, Integer value) {
    int i = 0;
    for(; i < node.children.size(); ++i) {
        PatriciaTrieNode child = node.children.get(i);
        int index = child.getEqualIndex(key);
        if(index == 0) {
            //new branch. child.key="bc", key="a"
            if(key.compareTo(child.key) < 0) {
                PatriciaTrieNode newNode = new PatriciaTrieNode(key, value);
                newNode.isTerminal = true;
                // maintain a lexi-order
                node.children.add(i, newNode);
                break;
            }
            // key is greater than all node's children
            if(i == node.children.size()-1) {
                PatriciaTrieNode newNode = new PatriciaTrieNode(key, value);
                newNode.isTerminal = true;
                node.children.add(newNode);
            }
            continue;
        }
        if(index == Math.min(key.length(), child.key.length())) {
            if(key.length() == child.key.length()) {
                if(child.isTerminal) {
                    //or throw exception, duplicate key
                    return;
                }
                child.isTerminal = true;
                child.value = value;
            }
            // child.key="abc", key="ab", index=2
            else if(key.length() < child.key.length()) {
                //split child
                PatriciaTrieNode splitNode = new PatriciaTrieNode(child.key.substring(index), child.value);
                splitNode.isTerminal = child.isTerminal;
                splitNode.children = child.children;

                child.key = child.key.substring(0, index);
                // key ends here
                child.isTerminal = true;
                child.value = value;
                child.children = new LinkedList<>();
                child.children.add(splitNode);

            }
            // child.key="ab", key="abc"
            else {
                // recursively insert the remaining key
                String remains = key.substring(index);
                insert(child, remains, value);
            }
        }
        // index < min length
        // child.key="abc", key="abde"
        // split both child.key and key
        else {
            //split child
            String subchildkey = child.key.substring(index);
            PatriciaTrieNode subchild = new PatriciaTrieNode(subchildkey, child.value);
            subchild.isTerminal = child.isTerminal;
            subchild.children = child.children;

            // split inserted key
            String subkey = key.substring(index);
            PatriciaTrieNode newChild = new PatriciaTrieNode(subkey, value);
            newChild.isTerminal = true;

            //update child
            child.key = child.key.substring(0, index);
            child.isTerminal = false;
            child.children = new LinkedList<>();
            // add children according to lexi-order
            if(subkey.compareTo(subchildkey) > 0) {
                child.children.add(subchild);
                child.children.add(newChild);
            }
            else {
                child.children.add(newChild);
                child.children.add(subchild);
            }
        }
    }
    // node has no child
    // directly insert a new child
    if(i == 0) {
        PatriciaTrieNode newNode = new PatriciaTrieNode(key, value);
        newNode.isTerminal = true;
        node.children.add(newNode);
    }
}
```

## 删除 

删除操作与插入操作，搜索操作的几个判断条件都是相同的，而根据Patricia的特性，只有一个孩子节点与它的父节点合并，删除节点会涉及到节点的merge。代码如下：

```java
private void delete(PatriciaTrieNode node, String key) {
    for(int i = 0; i < node.children.size(); ++i) {
        PatriciaTrieNode child = node.children.get(i);
        int index = child.getEqualIndex(key);
        if(index == 0) {
            if(key.compareTo(child.key) < 0) {
                // or throw no such key exception
                return;
            }
            if(i == node.children.size() - 1) {
                // or throw no such key exception
                return;
            }
            continue;
        }
        // index < min length
        if(index < Math.min(key.length(), child.key.length())) {
            // or throw no such key exception
            return;
        }
        // if(index == Math.min(key.length(), child.key.length()))
        if(key.length() == child.key.length()) {
            // key found
            // delete child, and merge nodes if necessary
            if(child.isTerminal) {
                //e.g. child="ab#", key="ab", node="a"
                //      a
                //     / \
                //    d  ab#
                if(child.children.size() == 0) {
                    node.children.remove(i);
                    // merge nodes for node
                    if(!node.isTerminal && node.children.size() == 1) {
                        mergeNodes(node, node.children.get(0));
                    }
                }
                else {
                    child.isTerminal = false;
                    // merge nodes for child
                    if(child.children.size() == 1) {
                        mergeNodes(child, child.children.get(0));
                    }
                }
            }
            else {
                // or throw no such key exception
                return;
            }
        }
        // child="abc#", key="ab", node="a"
        //     a
        //    / \
        //   a  abc#
        else if(key.length() < child.key.length()) {
            // or throw no such key exception
            return;
        }
        else {
            String remains = key.substring(index);
            delete(child, remains);
        }
    }
}

private void mergeNodes(PatriciaTrieNode parent, PatriciaTrieNode onlyChild) {
    parent.key += onlyChild.key;
    parent.value = onlyChild.value;
    parent.isTerminal = onlyChild.isTerminal;
    parent.children = onlyChild.children;
}
```

具体的流程不去分析了。

## 打印

打印函数能方便一些代码调试，给出一个简单的递归打印Patricia Trie的代码。

```java
public void printTree() {
    this.print(0, this.root);
}

private void print(int level, PatriciaTrieNode node) {
    for (int i = 0; i < level; i++) {
        System.out.format(" ");
    }
    System.out.format("|");
    for (int i = 0; i < level; i++) {
        System.out.format("-");
    }
    if (node.isTerminal) System.out.format("%s - %s #%n", node.key, node.value);
    else System.out.format("%s%n", node.key);
    for (PatriciaTrieNode child : node.children) {
        print(level + 1, child);
    }
}
```

完整代码可以参考GitHub仓库 - [DataStructure](https://github.com/MartyPang/DataStructures/tree/master/src/main/java/ecnu/dase/patriciatrie)

# 参考

[Wikipedia Radix tree](https://en.wikipedia.org/wiki/Radix_tree#Implementations)  
[PATRICIA Paper](https://dl.acm.org/citation.cfm?id=321481)  
