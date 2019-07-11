---
layout: post
title: '[LeetCode From Day One] - Tips and Tricks in Dynamic Programming'
author: Marty Pang
categories: 
  - Algorithm
tags: 
  - LeetCode
  - Dynamic Programming
last_modified_at: 2019-07-11T16:31:21-05:00
---

动态规划与回溯一样，作为五大常用算法之一，几乎是是笔试面试必考的内容。首先要明确的是，动态规划不是算法，而是设计算法的一种方法。DP解决的问题是一个多阶段决策问题，每阶段面临多个决策，选择最优决策达到下一状态，直至终态。DP则是在状态转移的过程中寻找最优解。本文总结动态规划问题的一些特征以及解决动态规划问题的过程。


* 目录
{:toc}


# 动态规划问题的特征

动态规划的核心是把原问题划分为子问题进行求解，这些问题有如下几个特征，包括最优子结构、子问题重叠、边界和无后效性。这几个看起来很高大上的名词，其实结合具体问题来看就是非常容易理解的。

1. 最优子结构
  在得到子问题最优解的前提下，母问题再通过一定的优化即可得到最优解。

2. 子问题重叠
  即子问题之间是不独立的，一个子问题的解可能在其他子问题求解过程中使用。这一点其实不算动态规划的必有的特征，但是如果子问题均是独立的，动态规划算法优势体现不出。例如在斐波那契的例子中，计算`F(4)=F(3)+F(2)`，得到子问题`F(3)`和`F(2)`。假设我们得到了子问题`F(2)`的 结果，那么在计算子问题`F(3)`时，直接使用其结果即可，无需重复计算。

3. 边界
  这个特性最容易理解，在一定时候不再需要提出子问题的情况叫做边界。例如斐波那契数列的边界为`F(0)`和`F(1)`。

4. 无后效性
  即某阶段的状态一旦确定，就不受之后的决策影响。换句话说，某状态至于当前状态有关，不影响之前状态。例如`F(4)`的计算并不会影响`F(0)`的结果。

## 模型
动态规划的经典模型有线性模型、区间模型、背包模型、状态压缩模型和树形模型。这部分内容可参考mmc2015的[动态规划总结](https://blog.csdn.net/mmc2015/article/details/73558346)，这里仅做简单引用。

1. 线性
  线性模型是DP中最常见的模型，线性指的是状态的排布是线性的，时间上或者空间上。

2. 区间
  区间模型的状态一般由`d[i][j]`表示区间`[i,j]`上的最优解，通过状态转移计算出区间`[i+1,j]`或`[i,j+1]`上的最优解。

3. 背包
  背包模型是DP中一个最经典的问题，包括01背包、完全背包、多重背包等，详细可参考[背包九讲](https://blog.csdn.net/yandaoqiusheng/article/details/84782655)。

4. 树状
  指状态转移图是一棵树，父节点的值通过所有子节点计算完毕得出。


# 解决动态规划问题

DP问题有固定的解题思路，一般包含以下几个步骤：
- 构造问题对应的过程，划分阶段；
- 确定每个阶段的状态以及各阶段之间传递的参数；
- 根据相邻两阶段的状态关系确定决策方式和状态转移方程；
- 确定边界，考虑边界的处理方式；
- 使用自顶向下或自底向上的方式计算最优值；

整个求解过程可以用一个二维的最优决策表来表示，行表示决策阶段，列表示问题状态，填表的过程就是找到`i`阶段`j`状态下的最优值，如背包问题`f[i][v]`表示前i件物品放入容量为v的背包可获得的最大价值。但是算法具体实现时，根据题目，可以创建一维数组或者二维数组。例如[最长子序列](https://leetcode.com/problems/longest-increasing-subsequence/)问题，只给出一个一维数组的，算法也可只定义一个一维数组。再比如[找零钱](https://leetcode.com/problems/coin-change/)问题，题目中给出了零钱面值总价值，就可以定义一个二维数组。值得注意的是，二维数组的解法可以用滚动数组优化为一维数组，后面的LeetCode题目会给出说明。

## top-down

自顶向下的做法其实是带记忆的递归解法（记忆化搜索）。也就是，我们首先考虑从最终状态出发，如果遇到一个子问题还未求解，那么就先求解子问题。一般用于解决树状模型的DP问题，从父节点出发找到所有的子问题，确定子问题之间传递的参数。仍旧以最简单的斐波那契为例，使用自顶向下的解法，代码如下：

```java
int fibonacci(int n, int[] f) {
    if(n == 0) {
        f[0] = 1;
        return f[0]
    }
    if(n == 1) {
        f[1] = 1;
        return f[1];
    }
    if(f[n] != 0) { // -1表示状态n未被计算过
        return f[n];
    } else {
        f[n] = fibonacci(n-1, f) + fibonacci(n-2, f);
        return f[n];
    }
}
```

## bottom-up

自底向上的做法简单来说就是根据递推关系，从初始状态开始逐步推导出最终状态。该方式一般用来解决能确定顺序的问题，例如线性模型从左往右转移，或是区间模型，从小区间推导到大区间。一个经典实现是斐波那契数列的递推关系，`f(n)=f(n-1)+f(n-2)`。

```java
int fibonacci(int n) {
    if(n == 0) return 0;
    if(n == 1) return 1;
    int[] dp = new int[n+1];
    dp[0] = 0;
    dp[1] = 1;
    for(int i = 2; i <=n ; ++i) {
        dp[i] = dp[i-1]+dp[i-2];
    }
    return dp[n];
}
```

# LeetCode动态规划题解

选取的题目涵盖线性模型，区间模型，一维数组解法，二维数组解法，初始状态推导与最终状态倒推。

## Longest Increasing Subsequence

最长增长子序列（LIS）的题目很简短，给定一个未排序的数组，求出最长的递增子序列。题目要求子序列，非子数组，子序列的元素不一定是连续的。例如给定的数组为`[10,9,2,5,3,7,101,18]`，则输出4，最长增长子序列为`[2,3,7,101]`。

首先我们对问题划分阶段，假设从左往右扫描数组，数组的每个小标表示一个阶段，例如阶段`i`表示找到子数组`[0,i]`的最长增长子序列。每个阶段的状态为LIS的长度，子问题为已知阶段`i`的状态（包括`i`之前），如何求出阶段`i+1`的状态。之后思考状态转移方程，我们定义一个一维数组dp[]，`dp[i]`表示阶段`i`的状态。对于阶段`i`的LIS长度`dp[i]`，我们遍历前`i-1`个阶段，找到最大的`dp[j]`，其中0<=j<=i-1，且nums[j]<nums[i]。即，状态转移方程为`dp[i]=max{dp[j]+1 | 0 <= j <= i-1 \& nums[j] < nums[i]}`。边界为`dp[0]=1`，即只有包含一个元素。

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums == null || nums.length == 0) return 0;
        if(nums.length == 1) return 1;
        int[] dp = new int[nums.length];
        dp[0] = 1;
        int prevMax = 0;
        int res = 0;
        for(int i = 1; i < nums.length; ++i) {
            prevMax = 0;
            for(int j = 0; j < i; ++j) {
                if(nums[j] < nums[i])
                    prevMax = Math.max(dp[j], prevMax);
                }
            }
            dp[i] = prevMax + 1;
            res = Math.max(dp[i], res);
        }
        return res;
    }
}
```

算法时间复杂度为$O(n^2)$，嵌套for循环。空间复杂度为$O(n)$。

## Longest Palindromic Substring

本题是动态规划区间模型的应用。我们定义状态`dp(i,j)`表示从i位置到j位置的字串是否为回文字符串。显然可以推导出，`dp(i,j)=(dp(i+1,j-1) and Si==sj)`，即如果dp(i+1,j-1)是回文串，并且i位置和j位置的字符相同，那么dp(i,j)也是回文字符串。

```java
class Solution {
    public String longestPalindrome(String s) {
        if(s.length() < 2) return s;
        int minIndex = 0, maxLen = 0;
        boolean[][] dp = new boolean[s.length()][s.length()];
        for(int i = 0; i < s.length(); ++i) {
            for(int j = 0; j <= i; ++j) {
                dp[j][i] = s.charAt(j) == s.charAt(i) && (i - j < 3 || dp[j+1][i-1]);
                if(dp[j][i] && i - j + 1 > maxLen) {
                    maxLen = i - j + 1;
                    minIndex = j;
                }
            }
        }
        return s.substring(minIndex, minIndex+maxLen);
    }
}
```

## Coin Change

[找零钱](https://leetcode.com/problems/coin-change/)，由于可重复使用某一面额的硬币，是一个完全背包问题。

> 给定几种不同面额的硬币coins和一个总数额amout，求出能组成amout的最少硬币数
> Example:  
&nbsp; &nbsp; Input: coins=[1,2,5], amount=11  
&nbsp; &nbsp; Output: 3

先给出二维数组的解法，再使用滚动数组去优化。

### 二维数组

题中有两个变量，硬币的面额与构成的总数额，故我们可以定义状态`dp[i][j]`表示给定面额为coins[0], coins[1], ..., coins[i]的硬币，使用多少个可以构成数额`j`。状态转移我们可以分三种情况讨论。
1. 第一行（`i==0`）
也就是只能使用面额为coins[0]的硬币，这种情况下，我们判断`j%coins[i] == 0`是否成立，若成立则填入`j/coins[0]`的值。

2. 第一列（`j==0`）
即待构成的总数额为0，所以第一列全部填0，即0张任何面额的硬币均能构成总数额0。

3. 其他（`i>0 && j>0`）
这种情况我们又可以分三种情况：
	- `coins[i]>j`，则直接使用上一行该列的值，因为`coins[i]`对构成j毫无帮助。`dp[i][j]=dp[i-1][j]`；
	- `coins[i]==j`，则直接填入1，因为只需使用一个面值为`coins[i]`的硬币即可；
	- `coins[i]<j`，则取`dp[i-1][j]`和`dp[i][j-coins[i]]+1`的较小者即可；
后两种情况其实可以合并为一种，当`coins[i]==j`时，一定是`dp[i][j-coins[i]]+1`最小，因为`dp[i][0] = 0`。
我们给出最终的实现代码：

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        if(coins.length == 0) return -1;
        if(amount == 0) return 0;
        int[][] dp = new int[coins.length][amount+1];
        for(int i = 0; i < coins.length; ++i) {
            for(int j = 0; j < amount+1; ++j) {
                dp[i][j] = amount+1;
            }
        }
        dp[0][0] = 0;
        for(int i = 1; i < amount+1; ++i) {
            if(i >= coins[0]) {
                dp[0][i] = Math.min(dp[0][i], dp[0][i - coins[0]]+1);
            }
        }
        for(int j = 1; j < coins.length; ++j) {
            dp[j][0] = 0;
        }
        for(int i = 1; i < coins.length; ++i) {
            for(int j = 1; j < amount+1; ++j) {
                if(j < coins[i]) { //面额大于amount
                    dp[i][j] = dp[i-1][j];
                }
                else { //面额大于等于amount
                    dp[i][j] = Math.min(dp[i-1][j], dp[i][j-coins[i]]+1);
                }
            }
        }
        return dp[coins.length-1][amount] >= amount+1 ? -1 : dp[coins.length-1][amount];
    }
}
```
算法的时间复杂度为$O(mn)$，其中`m`为硬币的种类，`n`为amount。空间复杂度为$O(mn)$。

### 一维数组

利用滚动数组，我们可以优化上述二维数组的解法。定义状态`dp[i]`为构成数额`i`所需要的最少的硬币数量，从`dp[0]`推导到`dp[amount]`。对于每一个状态，我们都遍历`coins`数组，判断每个硬币的使用能否得到一个更小的数量。

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        if(coins.length == 0) return -1;
        if(amount == 0) return 0;
        int[] dp = new int[amount+1];
        for(int i = 0; i < amount + 1; ++i) {
            dp[i] = amount + 1;
        }
        dp[0] = 0;
        for(int i = 1; i <= amount; ++i) {
            for(int j = 0; j < coins.length; ++j) {
                if(coins[j] <= i) {
                    dp[i] = Math.min(dp[i], dp[i-coins[j]]+1);
                }
            }
        }
        return dp[amount] == amount + 1 ? -1 : dp[amount];
    }
}
```

## Dungeon Game

[地牢游戏](https://leetcode.com/problems/dungeon-game/)为LeetCode困难难度174题。

> 地牢由M x N个房间组成dungeon[][]，其实K最初位于左上角的房间，骑士只能向右或者向下移动。每个房间有一个数值，表示K进入房间失去的血量或者是获得的血量，由数值正负决定。如果K的血量在任何房间降至0或以下，立即死亡。要求编写一个函数确定骑士一开始的最低血量。

![dungeon](/images/20190711/dungeon.png){:  .align-center}

本题的解法采用从最终状态往初始状态推导的方法，即骑士K在右下角房间，只能往上或者往左走。我们用`dp[i][j]`定义K在`[i,j]`处的血量，在任何坐标，dp至少为1。最终状态的血量为`max{1, 1-dungeon[i][j]}`。那么最后一列的值为`max{1, dungeon[i+1][j]-dungeon[i][j]}`，例如上图中`dp[1][2] = 5`，表示进入房间`dp[1][2]`时候，骑士至少还有5点血。最后一行的值同理为`max{1, dungeon[i][j+1]-dungeon[i][j]}`。而对于其余房间，因为我们是倒推的，对于房间`[i,j]`，骑士只能从`[i+1,j]`或者`[i,j+1]`来的。所以`dp[i][j] = max{1, min{dungeon[i+1][j]-dungeon[i][j],dungeon[i][j+1]-dungeon[i][j]}}`。最终返回`dp[0][0]`即可。具体实现不必定义新的dp数组，在原数组上该就行了。

```java
class Solution {
    public int calculateMinimumHP(int[][] dungeon) {
        if(dungeon == null || dungeon.length == 0 || dungeon[0].length == 0) return 0;
        for(int i = dungeon.length - 1; i >= 0; --i) {
            for(int j = dungeon[0].length - 1; j >= 0; --j) {
                if(i == dungeon.length - 1 && j == dungeon[0].length - 1) {
                    dungeon[i][j] = Math.max(1, 1 - dungeon[i][j]);
                } else if(j == dungeon[0].length-1) {
                    dungeon[i][j] = Math.max(1, dungeon[i+1][j] - dungeon[i][j]);
                } else if(i == dungeon.length-1) {
                    dungeon[i][j] = Math.max(1, dungeon[i][j+1] - dungeon[i][j]);
                } else {
                    dungeon[i][j] = Math.max(1, Math.min(dungeon[i+1][j], dungeon[i][j+1]) - dungeon[i][j]);
                }
            }
        }
        return dungeon[0][0];
    }
}
```

