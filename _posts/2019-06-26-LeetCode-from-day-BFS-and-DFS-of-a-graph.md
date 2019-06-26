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
    private int vNum; //节点数目
    private List<Integer>[] adj; //邻接表

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

广度优先搜索的时间复杂度为$O(V+E)​$，每个节点以及每条边都被访问一次。


# 深度优先搜索

DFS运用的其实是上一篇文章讲到的[回溯法](https://www.hytheory.com/algorithm/LeetCode-from-day-backtracking/)。它的基本步骤如下：
- 从图的某个节点`v`出发，首先访问`v`；
- 以`v`的第一个邻接节点`u0`作为新的节点，访问从`u0`出发的所有路径，回溯到`v`下一个未被访问的邻接节点；
- 重复步骤2；
- 若仍有节点未被访问（其他连通分支），则另选一个未被访问的节点重复以上3个步骤；

DFS的递归写法如下，同样的，我们需要一个`visited`数组保存所有节点的访问情况。

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

递归DFS的复杂度分析。首先递归深度为节点数目$O(V)$，采用邻接表表示图时，查找所有节点邻接点的时间复杂度为$O(E)$。故总的时间复杂度为$O(V+E)$。

非递归的DFS需要借助栈来实现，这里简单描述一下算法过程。i) 访问一个节点邻接的未访问，标记它，并且压入栈中；ii) 如果没有可访问的节点，则从栈中弹出一个节点；iii) 如果前两个步骤都无法执行，则完成DFS搜索。这与递归调用栈是一个道理。

# 拓扑排序

拓扑排序通常用来解决依赖图的问题。依赖图表示节点直接的依赖关系，比如数据库中的冲突依赖，表示并发事务之间的数据访问冲突，再比如先后依赖，先有鸡还是先有蛋，先起床再刷牙等等。拓扑排序就是根据这些依赖关系导出节点之间的一个顺序。值得注意的是，应用拓扑排序的图必须是有向无环图（DAG）。有环的图代表着循环依赖，两个节点代表的任务都无法顺利进行下去。对有向无环图G进行拓扑排序得到一个节点的线性序列$\mathcal{L}$，若$(u,v) \in E$，则u在$\mathcal{L}$中出现在v之前。

DFS与BFS均可以用来实现图的拓扑排序。

## BFS实现

BFS实现需要记录节点的入度信息，依次访问入度为0的节点，并且更新剩余节点的入度，直到所有节点被访问。按给定的边构造图的时候，记录每个节点的入度，在开始BFS之前，遍历indegree数组，将入度为0的节点先加入队列和拓扑排序结果。

```java
public void buildGraph(List<Integer>[] graph, int[] indegree, int[][]edges) {
    for(int i = 0; i < edges.length; ++i) {
        graph[edges[i][0]].add(edges[i][1]);
        indegree[edges[i][1]]++;
    }
}

public void topologicalSort(List<Integer> order, int vNum, int[][] edges) {
    List<Integer>[] graph = new ArrayList[vNum];
    for(int i = 0; i < vNum; ++i) {
        graph[i] = new ArrayList<>();
    }
    int[] indegree = new int[vNum];
    buildGraph(graph, indegree, edges);
    
    Queue<Integer> queue = new LinkedList<>();
    for(int i = 0; i < vNum; ++i) {
        if(indegree[i] == 0) {
            queue.offer(i);
            order.add(i);
        }
    }
    
    while(!queue.isEmpty()) {
        int neighbor = queue.poll();
        for(int n : graph[neighbor]) {
            if(--indegree[n] == 0) {
                queue.offer(n);
                order.add(n);
            }
        }
    }
}
```

## DFS实现

