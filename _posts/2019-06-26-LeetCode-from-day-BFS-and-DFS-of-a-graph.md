---
layout: post
title: '[LeetCode From Day One] - BFS & DFS of a graph'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - DFS
  - BFS
  - Graph
last_modified_at: 2019-06-26T16:31:21-05:00
---


图的广度优先搜索与深度优先搜索和之前总结过的[树](https://www.hytheory.com/algorithm/Leetcode-from-day-one-basic-data-structure-tree/)的类似。下图是一张简单的有向图。

![directed graph](/images/20190626/graph.png){:	.align-center}

图的表示方法有邻接表和邻接矩阵，本文用邻接表的方式来存储和表示图。对于包含n个顶点编号从0到n-1的图，我们可以用如下结构表示，其中数组下标表示节点编号。

```java
List<Integer>[] graph = new ArrayList[n];
```

图与树不同的地方在于图可能存在环路。在搜索过程中可能会访问到已访问的节点，我们用一个布尔数组表示节点是否已被访问过。

```java
boolean[] visited = new boolean[n];
```

本文使用的图定义如下。

```java
public class Graph {
    private int vNum;
    private List<Integer>[] adj;

    public Graph(int n) {
        vNum = n;
        for(int i = 0; i < n; ++i) {
            adj[i] = new ArrayList<>();
        }
    }
}
```

# 广度优先搜索

广度优先指的是从一个节点出发，先访问该节点的所有子节点，然后遍历每个子节点，访问子节点的子节点。一般用队列保存节点的子节点，节点`v`从队列中出队，`visited`标记为true之后，将`v`的所有邻居节点入队。

![directed graph](/images/20190626/bfs.png){:	.align-center}

如上图，假设从节点2开始进行广度优先搜索。首先我们访问节点0和节点3，`visited[0]`和`visited[3]`标记为true。之后再访问节点0的邻居——节点2和节点1。我们发现节点2已经被访问过，只需访问节点1。所以最终的搜索顺序为`2,0,3,1`。

```java
public void BFS(int v) {
    boolean[] visited = new boolean[vNum];
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(v);
    visited[v] = true;

    while(!queue.isEmpty()) {
        int s = queue.poll();
        System.out.print(s + " ");
        for(int p : adj[s]) {
            if(!visited[p]) {
                queue.offer(p);
                visited[p] = true;
            }
        }
    }
}
```

注意到上述的代码只能遍历从节点v出发可达的节点，若想对所有节点做BFS，写个for循环即可。`visited`与`queue`对每次bfs应该是全局的。

```java
public void BFSTraverse() {
    boolean[] visited = new boolean[vNum];
    Queue<Integer> queue = new LinkedList<>();
    for(int i = 0; i < vNum; ++i) {
        if(!visited[i]) {
            queue.offer(i);
            visited[i] = true;
            while(queue.isEmpty()) {
                int s = queue.poll();
                System.out.print(s + " ");
                for(int p : adj[s]) {
                    if(!visited[p]) {
                        queue.offer(p);
                        visited[p] = true;
                    }
                }
            }
        }
    }
}
```

广度优先搜索的时间复杂度为$O(V+E)$，每个节点以及每条边都被访问一次。


# 深度优先搜索

```java
private void dfsHelper(int v, boolean[] visited) {
    visited[v] = true;
    System.out.print(v + " ");
    for(int p : adj[v]) {
        if(!visited[p]) {
            dfsHelper(p, visited);
        }
    }
}

/**
 * Run a DFS traverse for all nodes
 */
public void DFSTravesal() {
    boolean[] visited = new boolean[vNum];
    for(int i = 0; i < vNum; ++i) {
        if(!visited[i]) {
            dfsHelper(i, visited);
        }
    }
}
```

# 拓扑排序

## BFS实现

## DFS实现
