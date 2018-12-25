---
layout: post
title: '可串行化快照隔离'
description: "Serializable Snapshot Isolation 可串行化快照隔离"
date: 2018-12-25
author: MartyPang
cover: '/assets/img/SSI/ssi-cover.jpg'
tags: [Serializable, Snapshot, Isolation, 可串行化, 快照隔离, 数据库]
---

> 可串行化快照隔离

# Serializable Snapshot Isolation

## 简介
可串行化是保证事务正确执行不破坏一致性约束的一个重要属性。为了实现可串行化，数据库用户（DBA或者编程人员）可以在应用层编码实现事务间的隔离。但是这种方式有两个缺点，其一是非通用，为应用量身定制的，移植性差；其二是大大增加编码人员的工作量，数据库易用性差。如果数据库本身提供保证所有事务执行都是可串行化的机制，那么开发人员无须担心并发事务破坏数据一致性的问题。一些并发控制协议如两阶段锁能保证事务的可串行化，但是遭受性能的问题。快照隔离继承了多版本的优势，使得读操作永远都不会被阻塞，总能读到一致版本的数据。SI 相较于2PL有着更高的吞吐，特别 对一些read-heavy的负载。但是在SI提出的时候就允许非可串行化的调度出现，并且这种异常非常普遍。在本文之前，有着许多工作研究如何在SI基础上保证可串行化。主要的技术是分析应用程序代码，检测可能发生的异常，通过修改代码来保证可串行化。正如之前说的，这种方式对开发人员非常的不友好。
本文在数据库层面提供一套机制保证任意事务执行的可串行化，并且有着与SI相提并论的性能。本文的贡献有二，其一是提出了一个易于实现的算法保证任一执行都是可串行化的，并且读操作永远不被阻塞；其二是一定条件下，该算法的吞吐接近于SI。
算法的主要思想是在事务运行时动态的检测某一特定的冲突模式（后面介绍，称为dangerous structure），然后中止相关事务来避免异常发生。算法与基于冲突图检测的算法相似，不同的是，本文提出的算法并不是在commit-time检测并中止事务，而是在运行时实时检测并提前中止事务；同时，该算法不需要任何的环检测算法，只需检测某一特定的结构。

## 背景
### 隔离性
在事务型数据库系统中，事务的隔离性指的是，多个用户或者多个进程并发访问并操纵数据库时，他们发起的并发事务之间是隔离的互不影响的。换句话说，若能保证事务之间完全的隔离性，即可并发事务之间的执行结果与某个串行的调度的执行结果是一致的。这样的调度我们称为一个正确的调度。事务调度的正确性是事务系统中的一个基本问题。在保证事务调度正确性的前提下，随之而来的一个问题是，如何最大限度地提高事务的并发度，即提高数据库性能。
传统数据库出现以来，人们针对上述两个问题做了许多的研究工作。ANSI标准尝试定义一个与实现无关的隔离级别。ANSI标准是基于表现出来的异常现象（phenomena）来定义隔离级别。这个想法很好，但是Phil Bernstein和Jim Grey等大牛表示这个标准有点问题。其一是这种定义是通过自然语言描述，不够严谨；其二是后续有多了MVCC等技术，隔离级别的定义也需要相应改动。于是几个大牛写了A Critique of ANSI SQL Isolation Levels这篇经典的论文，在ANSI标准基础上重述几个并发异常，并补充了mvcc技术出现后新的并发异常。图1是Critique论文重述之后定义的隔离等级。

![full_isolation](/assets/img/SSI/full_isolation.png)

并发异常的定义不再赘述，详见[1]。

可以看到，可串行化隔离等级避免了所有的并发异常，我们认为可串行化的调度是一个正确的调度。到目前（论文发表时间2008年）为止，避免所有异常，或者说，保证可串行化隔离等级的唯一方法是引入锁机制。最常用的机制是1976年Eswaran提出的两阶段锁（2PL）机制[2]。2PL大致的一个思路就是，事务执行前先获得需要的所有读写锁，再事务释放一个锁之后就不能再去获得任何锁。Eswaran提出2PL定理，如果数据库系统的所有事务都遵循2PL机制，那么产生的调度就是可串行化的。2PL从一些方面来看很好，DB2，MS SQL Server，MySQL等数据库都使用2PL来保证可串行化隔离等级。但是2PL的性能依赖于数据库负载的读写比例。对于一些高冲突高竞争的负载，2PL有着不错的吞吐量。但是一些读事务比例较高的负载，使用2PL的数据库系统的事务中止或者阻塞的几率很高。在2PL下，只读事务也需要去获得读锁，会被写操作阻塞。在OLAP的数据库系统，有大量的只读事务，2PL浪费时间浪费资源。另外2PL也存在死锁现象，数据库系统需要额外设计死锁检测的机制。

### 快照隔离（Snapshot Isolation）
MVCC的介绍这里不再赘述，详见简介页面。

快照隔离[3]是MVCC技术的一种实现方式。SI同样继承了MVCC的好处：非阻塞的读操作。 当一个事务T开始执行时，它得到一个逻辑时间戳start-time（T）; 每当T读取一个数据项x时，不一定会看到写入T的最新值; 而是看到在T启动之前提交的事务中最后一个提交的x的版本（也有一个例外：如果T自己修改了x，它会看到它修改的版本）。 
SI在事务执行时会强制做一个额外的检查，称为First-Committer-Wins（FCW）规则：两个修改同一数据对象的并发事务不可能同时成功提交。实际实现中，SI会提前中止两个事务中的一个。
SI最早在[3]中提出，目前已经在Oracle，PostgreSQL，SQL Server 2005和Oracle Berkeley DB等数据库中实现。SI具备MVCC的优势，性能上远远优于2PL，与此同时，SI也避免了许多并发异常，诸如Lost Update和Inconsistent Read。Oracle与9.1版本之前的PostgreSQL称其实现的SI为可串行化隔离等级。

### Write Skew
但事实上，SI并不能保证所有的调度都是可串行化的。当两个并发事务的修改没有交集，这样的修改却会破坏一些约束，但是在SI下两个事务都能成功提交。这种异常称为write skew。一个简单的例子说明什么是write skew异常。假设有两个事务，T1和T2，从两个银行账户x和y取钱，事务执行必须不能破坏x+y>0的约束。现在有如下所示的执行过程：
             r1(x=50,y=50)r2(x=50,y=50)w1(x=-20)w2(y=-30)c1c2
T1和T2两个事务在SI下均能成功提交，但是最后x+y=-50，这破坏了约束，即发生了异常。这种异常就称为write skew。

## 可串行化快照隔离
下面介绍论文的主要工作，Serializable Snapshot Isolation（SSI）算法的实现。

### Multi-version Serializable Graph
Adya在MIT的博士毕业论文[4]中提出多版本冲突串行化图（MVSG）。MVSG是一种有向图，用来刻画事务之间的依赖（冲突）关系。每个节点代表一个已提交事务，事务之间有一条或者多条有向边连接。有向边有三种类型，分别是：
- ww-dependencies：已提交事务T1产生数据对象x的一个版本，提交事务T2在该版本基础上产生x的下一个版本，则T1有一条指向T2的ww边；
- wr-dependencies：提交事务T1产生x的一个版本，T2读了此版本（或者更新的版本），则T1有一条指向T2的wr边；
- rw-dependencies：提交事务T1读了x的一个版本，T2在该版本的基础上产生了x的一个新版本，则T1有一条指向T2的rw边；

图2为MVSG的一个示例，代表是上部分介绍的write skew异常。

![mvsg](/assets/img/SSI/mvsg.png)


### Dangerous Structure

我们知道，一个无环的MVSG一定是冲突可串行化的。Adya进一步提出，SI下产生的MVSG中的环必定包含连续两条rw边。论文讲两个并发事务之间的rw边称为vulnerable edge，进一步，连续两条vulnerable egde构成的结构称为dangerous structure。图3为dangerous structure的一个示例。

![ds](/assets/img/SSI/dangerous structure.png)

### 算法
论文提出的算法的主要思想就是，所有的事务按照SI执行，但是DBMS提供额外的机制，动态地检测非可串行化执行的发生，然后中止相应的事务。作者在设计算法考虑到这样一个tradeoff：如果检测尽可能少的情况，那么一些非可串行化的执行仍然存在，仍然不能保证可串行化（这违背了算法的初衷）；如果检测过多的情况，那么意味着会有更多的abort出现，性能会很糟糕。首先能想到的一个naive的算法就是去检测环，中止使得MVSG成环的事务。这种算法的开销很大，每次操作都需要O(N^2)的代价去检测是否成环。作者就想，是否可以在有少量不必要的中止的前提下，降低检测算法的开销，并且同样保证可串行化。
在Adya和Fekete的研究基础上，作者提出dangerous structure结构，并基于此实现了开销远远小于环检测的算法。算法动态去检测dangerous structure，一旦检测到就中止相应的事务。为了支持该算法，DBMS需要为每个事务维护两个布尔变量：T.inConflict，表示是否有指向T的rw依赖；T.outConflict，表示T是否有指向其他事务的rw依赖。当某个事务的T.inConflict和T.outConflict都为true时，表示算法检测到了一个dangerous structure。
算法修改了事务处理的四个操作（begin，read，write，commit）的代码实现dangerous structure的实时检测。图4 - 图7分别是修改的代码。

![4](/assets/img/SSI/4.png)

事务的begin操作初始化T.inConflict和T.outConflict为false。

![5](/assets/img/ssi/5.png) 

事务T在执行读操作时，首先要获得读取数据的SIREAD锁。如果数据对象x上有写锁，说明T与拥有该写锁的事务之间有一条rw依赖。随即设置T.outConflict为true，拥有写锁的事务的inConflict为true。接下来再去检测是否有连续第二条rw依赖。算法会去检测所有产生比T读取数据版本更新数据的事务的outConflict是否为true，若果为true，那么abortT。

![6](/assets/img/SSI/6.png)

事务写操作的代码与读操作类似，同样是去检测事务是否构成了dangerous structure。

![7](/assets/img/ssi/7.png) 

事务commit阶段，算法检测T.inConflict和T.outConflict是否均为true，若是，abort该事务。


### 正确性保证与假阳现象
本小节介绍SSI算法正确性的保证。Fekete在2005年一篇论文[5]中提出并证明这样一个定理：任何非可串行化的执行中必定存在一个dangerous structure。又作者提出的算法检测每一条rw依赖边。上一小节设计的算法也表明在事务做读操作，写操作和提交操作时，算法都会去检测是否存在连续两条rw依赖边并中止pivot事务破环。这也保证了该算法下所有的事务执行都是正确的。

该算法也存在一个弊端，就是会做一些不必要的中止，即存在假阳现象。举个例子来说，图8中，事务T0在执行w0(x)操作时，根据算法，T0.outConflict会被设置为true。事务T1在执行w1(y)操作时，T0.inConflict会被置为true。在T0提交时，算法检测到out和in均为true，于是abort T0。但事实上，图8的执行与串行化执行{TN, T0, T1}是等价的，因为T1并没有到TN的一条依赖边，也就是不存在环。

![eg](/assets/img/SSI/eg.png)

## 性能
本小节介绍该算法的性能实验。
作者在Berkeley DB中实现了SSI算法。Berkeley DB是一个开源的嵌入式数据库，本身支持SI和S2PL实现的可串行化隔离等级。
为了验证SSI的效果，作者选取了在SI下会发生并发异常的benchmark，SmallBank。SmallBank是对银行转账存钱应用的一个简单建模。其各个事务的依赖关系如图9所示。

![smallbank](/assets/img/SSI/smallbank.png)

可以观察到，图中存在连续两条rw边，分别是Bal --> WC，和WC --> TS，并且，TS有一条指向Bal的wr边，这也就构成了环。

### 短事务
短事务的实验结果图如图10与图11所示。

![short1](/assets/img/SSI/s1.png)

![short2](/assets/img/SSI/s2.png)

图10中横轴是并发访问数据库的线程数，纵轴是吞吐。图11中纵轴是错误比率。从图10中我们可以看出，SSI的性能要明显好于S2PL，在MPL为20时，SSI的性能近乎是S2PL的10倍。这是因为S2PL中的读写操作互相阻塞，并且冲突是通过死锁检测发现的，时延更长。图11表明虽然SSI的错误率比S2PL和SI都略微高了一点，但是其大部分的错误都是检测到dangerous structure而不是冲突。

### 长事务

![long1](/assets/img/SSI/l1.png)

![long2](/assets/img/SSI/l2.png)

长事务的实验结果中，快照隔离的性能在并发线程数大于10之后，明显的好于2PL，SSI的性能与SI的性能相较无差别。

## 总结
本文提出了一种基于快照隔离方式实现可串行化隔离等级的新方法。该算法的原型已经在Oracle Berkeley DB中实现，并且实验结果表明在各种情况下SSI的性能明显优于S2PL，并且与SI相近。
SSI算法在某些情况下存在比SI更高的中止比例。未来的工作方向是研究是都存在一种方法在保有性能的前提下降低false positive的比率。

## 参考文献
[Seriliazable Isolation for Snapshot database](https://dl.acm.org/citation.cfm?id=1620587)