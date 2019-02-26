---
layout: post
title: 'SQL on Hadoop Survey'
description: "在Hadoop上实现SQL功能综述"
author: Marty Pang
image: 
  path: /images/20180215/sql-on-hadoop.jpg
  thumbnail: /images/20180215/sql-on-hadoop.jpg
categories: 
  - BigData
  - Database
tags: 
  - SQL
  - Hadoop
  - Survey
  - 综述
  - NoSQL
last_modified_at: 2018-02-19T22:12:29-05:00
---

# SQL on Hadoop综述

## 简介

### 大数据、Hadoop和SQL
近年来，数据科技产业发展趋势迅猛，社会信息化进程不断加速。人们的信息采集渠道日益丰富，信息应用面不断拓宽，信息总量也以指数级上升。图1-1显示了2017年1分钟的互联网时间内产生的数据量，而这仅是冰山一角。如何有效应对海量数据带来的管理与应用的难题已经成为信息行业乃至整个社会亟需解决的问题。

Hadoop[1]包含通用的并行计算框架MapReduce和底层的分布式文件存储系统HDFS。MapReduce是用以处理大数据集合的编程模型。MapReduce框架可以把一个应用程序分解为许多并行的map任务，跨大量的计算节点运行非常巨大的数据集，计算结果发送到reducer上进行加载合并等处理。HDFS最早是Google文件系统的开源实现。HDFS具有高容错性的特点，并且被设计用来部署在低廉的机器上。在HDFS上跑的应用通常需要访问超大的文件集合，HDFS提供以流的形式访问数据。这大大提高了数据传输访问的吞吐量。存储在HDFS上的数据通常在不同的节点上有1至2份备份，一台机器的宕机并不会引起系统故障。Hadoop高可靠性、高容错性、强可扩展性和高效性等特性使其收到众多互联网公司的青睐，也引起了学术界的大量关注。目前为止，Hadoop技术在互联网领域，包括Facebook、Yahoo等，已经得到了广泛的应用。

Hadoop有着众多大数据处理方面优秀的特性，但是Hadoop也存在局限性。第一，Hadoop使用门槛很高，除了专门的编程人员以及系统管理员，很少有其他用户可以上手使用Hadoop来完成一些数据访问以及分析的任务。第二，Hadoop并不能及时响应分析型的任务。如果一些互联网应用想在Hadoop上做查询，等待时间将是灾难性的。第三，Hadoop并不是为高用户并发而设计的。总的来说，Hadoop对大部分的互联网应用并不友好。

SQL是访问和操纵数据的工具，它在互联网领域无处不在。它不再是仅供开发人员和DBA和分析人员使用的工具。如今，大量商用产品和应用程序使用SQL来查询，操纵和可视化数据。用户可以很容易地使用SQL来做数据查询以及数据更新等任务。

![图1-1](/images/20180215/this-is-what-happens-to-internet-every-minute.jpg){:  .align-center}
**图1-1 互联网时间一分钟**

### 为什么需要在Hadoop上实现SQL

互联网企业在选择大数据处理工具时，必须要权衡现有的使用SQL实现的工具和移植到Hadoop平台的利弊。
在大数据时代，传统的针对事务型、操作型以及分析型负载的工具在性能、扩展性和容错性上表现十分糟糕，具体表现在响应时间慢，缺乏敏捷性，缺乏处理多种数据类型的能力。而Hadoop能够灵活地灵活处理任何数据类型，包括结构化，半结构化与非结构化数据。另外，Hadoop天然地可以将任务并行化，提高处理效率。但是与此同时，用户通常需要经过专业的训练才能使用Hadoop提供的一些API进行开发。这是一个陡峭的学习曲线，并且需要耗费大量时间编写大量的代码。Hadoop的架构设计使得数据存储和数据访问之间产生了不匹配。Hadoop底层存储HDFS满足大数据存储，满足强扩展性，高容错性。但是在HDFS上仅仅做一些MapReduce并行计算似乎有些浪费。在如此大数据量的存储上能够实现标准SQL的功能，这对许多公司甚至整个行业都是非常有吸引力的一件事。
在Hadoop上实现SQL这件事，无论是工业界还是学术界，一直都有人在研究。2009年，Hive的提出是一个里程碑式的事件。在Hive之前没有一个工具或者系统可以在Hadoop上实现SQL。2009年至今，已经出现了几十种商用的和开源的产品在Hadoop上实现SQL功能。图1-2是该类产品出现的时间线。

![图1-2](/images/20180215/SQL-on-Hadoop/timeline.png){:  .align-center}
**图1-2 SQL-on-Hadoop解决方案概览**

## 挑战与设计目标

一个SQL-on-Hadoop的解决方案视其目的不同有许多的设计目标，其中就包括实现传统RDBMS中SQL 的功能。本文主要从AP角度分析SQL-on-Hadoop的解决方案。
一个SQL-on-Hadoop方案的设计需要考虑的设计目标主要有以下四点。
1. 可扩展性强 。SQL Server和MySQL等数据库在不大量改动代码和编写sharding策略的前提下很难实现扩展。而像Oracle[2]和DB2[3]这种shared disk的数据库的横向扩展本身就很昂贵。引入Hadoop就是为了克服传统事务数据库扩展性弱的缺点。
2. 考虑应用能接收的时延，尽可能地降低查询时延。设计者需要考虑如何降低延迟时间和 I/O。
3. 高并发用户访问。Hadoop在处理并发用户方面并没有做得很好，所以设计者需要设计资源协调模块进行资源的分配与调度。
4. 可以做到query in place，而不是将数据从HDFS抽取出并持久化到另外的系统进行处理。
5. 尽可能多地支持标准SQL。如果可以，设计者应当100%实现ANSI SQL。

## 解决方案

在开发Hadoop的SQL解决方案领域中，无论是开源社区还是企业都开发了大量的工具，使得SQL可以在Hadoop平台使用。开源社区贡献的工具包括Hive、Apache Drill、Impala、Tajo、Spark SQL等。商用工具包括Citus、HAWQ、JethroData、Microsoft PolyBase、Teradata、Vertica等。这些产品要么是开发了一套全新的SQL引擎，要么是沿用RDBMS中一些经典的思想或技术。但是这些产品有一个共性，那就是致力于实现大数据集的处理和系统的水平扩展。

现有的SQL-on-Hadoop产品或工具根据技术的不同可以分为以下四种类型：
- 开发一个SQL转换层可以把SQL转换成等价的MapReduce代码并在集群上执行。Apache Hive就是使用该技术的一个最好的例子。Hive使用MapReduce和Tez作为中间处理层，具体的架构会在3.1节详细介绍。
- 沿用关系型数据库成熟的技术。RDBMS自上世纪70年代提出以来，经过40多年的发展，其存储引擎与查询优化等技术已经十分成熟。一个使用RDBMS实现SQL-on-Hadoop的例子就是集群的每个datanode都内置一个MySQL/Postgres引擎，并在RDBMS引擎和datanode之间创建一个中间件。内置的RDBMS引擎通过该中间件与datanode进行通讯，读取HDFS的数据并转换成RDBMS可以处理的数据格式。使用该技术实现SQL-on-Hadoop的产品有Citus和HAWQ。
- 造一个新的查询引擎可直接在HDFS存储的数据上执行SQL。Apache Drill与Impala使用了该技术，并采用多种优化是的交互式的SQL查询成为可能。
- 使用已有的分析数据库（部署在与Hadoop不同的集群上）与Hadoop集群的datanode交互。分析数据库集群使用connector从HDFS获取数据，在自身集群执行SQL查询。有一些原本就是做分析数据库起家的企业在实现SQL-on-Hadoop时会选取该种方式，例如Vertica和Teradata。

当然也可以根据用户更关心的查询时延长短，可以将SQL-on-Hadoop系统分为三类：
- Batch SQL，Batch SQL 的查询时间通常在分钟，小时级别，一般用于复杂的 ETL （Extract，Transform，Load）处理，数据挖掘，复杂数据分析等任务。Batch SQL 中最典型的系统是 Hive。Spark SQL 也可以归类到该系统。
- Interactive SQL， 也叫做交互式 SQL 查询，用户通常在同一个表上反复的执行不同的查询，Interactive SQL 的查询时间通常在毫秒级或者秒级以内，一般不超过分钟级别。该类系统主要追求低延迟Interactive SQL 在实现上通常采用 MPP 架构，并且将热点数据缓存到内存中，比如 Presto，Impala，Drill，HAWQ。鉴于 Spark SQL 也具有非常高效的查询速度，Spark SQL 也可以归类到 Interactive SQL 中。
- Operational SQL, 通常是单点查询，延时要求小于 1 秒，该类系统主要是 HBase。

本节以查询时延长短作为分类标准详细介绍不同类型的SQL-on-Hadoop解决方案。

### 批处理 SQL

本小节详细介绍Hive的架构以及它是如何将SQL转换成MapReduce任务的。Hive[4]是第一个在Hadoop上实现SQL功能的系统。Hive本质上是一个批处理系统，它更多的关注于系统的吞吐量而不是延时。Hive难以满足商业应用交互式查询的时延要求，但是近几年的版本也做了许多优化来缩短查询时延。后面的章节会介绍这些优化手段。

Hive最初是由Facebook于2009年开发的。其开发的一个主要目的就是使业务分析员能够使用他们熟悉的SQL语言访问存储在HDFS上的数据。在当时，访问HDFS存储的数据的唯一方式是使用MapReduce和Hadoop的API，而这些是需要相当的学习成本的。所以Hive应运而生。Hive是一个建立在HDFS上层的SQL引擎，它内部使用了MapReduce。Hive允许用户使用HQL（一种类SQL语言）查询HDFS的数据。如上所述，Hive最初的设计目的不是一个交互式的查询引擎，它是一个批处理系统，查询的结果通常会在几秒内返回，对于复杂查询，该时延可能以分钟计，实时性很差。但是随着Hadoop被更多的企业采用，用户对Hive的能力和性能也有了更多的需求。

图3-1为Hive的整体架构设计。

![](/images/20180215/hive architecture.jpg){:  .align-center}
**图3-1 Hive架构**

用户可以通过CLI或者web界面发送query请求，也可以使用JDBC或者ODBC connectors编写程序访问Hive。Hive还提供Thrift接口（Facebook开发的用作系统内部各编程语言之间的RPC通信），用户可以采用不同编程语言编写客户端访问。Hive Driver处理所有的query请求，管理该请求的整个生命周期和连接的session信息。编译器，优化器和执行器是Hive的核心部分，分别负责HQ的解析、语法检查和执行计划的生成，查询计划优化和MR任务的执行。Hive的metastore组件使用关系数据库，MySQL或者Postgres，来存储元数据，该关系型数据库通常作为单实例部署在Hadoop集群上。元数据包括：数据库，数据库属性，表，视图， 列的数据类型，表的文件格式，文件位置，分区，桶等信息。

编译器产生的查询计划是一个有向无环图（DAG），每个节点代表一个stage，每个stage表示一个map/reduce任务或者一个对元数据或者HDFS的操作。执行器会将收到的执行计划发送给YARN，由YARN分配任务给各个datanode进行执行。Hive的metastore组件使用关系数据库，MySQL或者Postgres，来存储元数据，该关系型数据库通常作为单实例部署在Hadoop集群上。元数据包括：数据库，数据库属性，表，视图， 列的数据类型，表的文件格式，文件位置，分区，桶等信息。

接下来我们通过两个例子说明Hive是如何把Hive QL转换成MapReduce任务的。首先看一个最基本的SQL语句：
```sql
SELECT col1, col2 from t1 where col1 = 4;
```
这条query首先会扫描t1表所有的行，选出col1为4的行，接着做一个投影，返回投影结果。这是一个只包含map的查询，mapper根据where语句选择出所有符合条件的行，然后输出col1和col2。mapper的伪代码如下所示：
```java
Map(k, record)
{
  if(record.col1 == 4){
    outRecord = <col1, col2>; //符合选择条件的行做一个投影操作，创建一条新元组
    
    collect(k, outRecord);    //输出key相同的元组
    
  }
}
```

再来看一个包含聚合函数的SQL语句在Hive中是如何转换成MapReduce任务的。
```sql
SELECT col1, sum(col2), avg(col3), from t1 groupby col1
```
这条SQL语句包含sum和avg两个聚合函数，所以与上一个例子不同，本例既包含mapper也包含reducer。该例子的map函数与上一个例子很相似，不同的是map输出记录的key应该为col1，因为按col1进行分组。reduce函数计算所有key相同的记录的sum(col2)和avg(col3)。伪代码如下：
```java
Map(k, record)
{
  outRecord = <col2, col3>;
    
  collect(col1, outRecord); //groupby col1
}

Reduce(k, listOfRecords)
{
  sum = avg = 0;
  foreach record in listOfRecords{
    sum += record.col2;
    avg += record.col3;
  }
  
  avg = avg/listOfRecords.length;
  outRecord = <sum, avg>;
  emit(k, outRecord);
}
```

Hive最初只支持MapReduce执行框架，这种批处理的方式确实能解决很多实际应用，但对于诸如多个MapReduce任务链在一起的负载，下一个MR任务需要使用上一个MR任务的结果时，Hive的性能很糟糕。之后Hive的更新版本支持了Apache Tez[5]引擎。Tez是一个运行于Yarn之上支持DAG作业的计算框架。Tez将Map和Reduce操作进一步拆分成更细的操作，操作之间可以灵活组合形成新的操作，这些操作经过一系列的组装形成一个DAG作业。Tez可以有效减少对HDFS的读写。我们将用一个例子来说明Tez是如何减少对HDFS的读写的。考虑有这个一个SQL：
```sql
SELECT Table1.col1, Table1.average, Table2.count from (SELECT col1, avg(col2) from X groupby col1)Table1 join (SELECT col1, count(col2) from Y groupby col1)Table2 on (Table1.col1 = Table2.col2)
```
图3-2左半部分显示的是该SQL直接被转换成MapReduce任务的物理执行计划。我们可以看到，groupby的中间结果是需要写回HDFS的。join的时候再次从HDFS读取该结果。当Hive采用Tez作为执行引擎的时候，执行计划就转变为了图3-2右半部分。从图中可以看到，Tez消除了额外的磁盘读写，这大大简短了一个query执行的时延。

![](/images/20180215/join eg2.png){:  .align-center}
**图3-2 join例子**

除了接入Tez引擎，Hive开发人员也做了其他优化手段来降低时延和I/O，包括：
- 向量化查询；
- 使用基于开销的查询优化器（CBO）；

### 交互式 SQL
大数据的交互式查询主要用于支持数据发现，数据挖掘以及复杂的数据分析功能。商业智能（BI）和可视化分析工具正在不断发展，以便与大数据生态系统交互式地工作。交互式的工作负载包括通过前端工具，API或命令行对数据执行特定SQL查询的能力。这些交互式的SQL引擎使用各种优化技术执行即时查询，以便在最短的时间内返回结果。
Hadoop上实现交互式SQL有两个关键的挑战，第一是更低的时延，query的响应时间相比于批处理SQL应该有显著的缩短。第二是兼容ANSI SQL，更好地支持BI工具。关于大数据的交互式SQL引擎是一个激烈竞争的领域，商业和开源产品都提出了创新的想法和架构来应对挑战。 一些比较流行的交互式SQL引擎包括：Spark SQL，Impala，Apache Drill，Jethro和Vertica。本章将重点介绍Impala和Vertica。Impala和Vertica均使用MPP架构实现SQL-on-Hadoop。MPP 架构的优点是查询速度快，通常在秒计甚至毫秒级以内就可以返回查询结果，这也是很多强调低延迟的系统采用 MPP 架构的原因。

#### Impala
Impala[6]的实现方式在技术上属于第三类，即造一个新的查询引擎可直接在HDFS存储的数据上执行SQL。Impala使用其独特的分布式查询引擎来缩短响应时间。 该分布式查询引擎安装在群集中的所有datanode上。Impala有两个设计理念：其一，就是Impala一定要很快，快到能支持交互式SQL查询，也就是时延在毫秒级；其二是确保查询引擎的线性可扩展性。Impala优化CPU使用率，并且大量利用现代CPU指令快速生成代码，以在最短时间内获得对大数据集的查询结果。

##### Impala架构
本小节中我们重点介绍使Impala成为交互式SQL的几个重要组件。图3-3为Impala的一个架构图。

![](/images/20180215/impala.jpg){:  .align-center}
**图3-3 Impala架构**

Impala包含三个主要的守护进程（长时间运行的进程），用于处理Impala查询所需的功能。 Impala守护进程运行在每个datanode中， 这是在Impala的安装阶段完成的。 Impala守护进程在要查询数据的同一节点上运行，保留数据访问的局部性。 每个Impala守护进程都是share nothing体系结构中的独立的进程。
与客户端（例如，impala-shell）连接的Query Planner作为主查询计划者，而其他节点充当查询执行引擎。 换句话说，一个Impala守护进程充当主，而在每个Hadoop datanode上运行的其他Impala守护进程充当执行引擎。 以下概述了Impala引擎的主要组件。

- impalad：与DataNode运行在同一节点上，由Impalad进程表示。其功能是使用引擎中构建的所有优化规则处理查询，以最有效的方式访问和处理数据。 Impala在很大程度上依赖于HDFS的数据分布，以确保数据访问的局部性。 impalad进程有三个组件：查询计划（Query Planner）进程，查询协调（Query Coordinator）进程和查询执行（Query Executor）进程。 Query Planner检查SQL语句的语法和语义，将其转换为逻辑和物理计划，最后将计划编译成由查询协调器（Query Coordinator）执行的分布式物理查询计划。 impalad直接从datanode本地磁盘读取数据。，这最大限度地减少了网络负载，提高了数据访问的局部性。
- statestored：维护运行在datanaode上的Impala守护进程的状态，其中状态包括有关节点运行状况的信息。 该守护进程监视群集中所有节点上impalad的健康状况。 如果一个impalad守护进程挂了，statestore守护进程将与集群中的其他节点进行通信，并且之后的查询不会发送到无响应的impala节点。 这个守护进程只有一个运行的进程。
- CLI: 提供给用户查询使用的命令行工具（Impala Shell使用python实现），同时Impala还提供了Hue，JDBC， ODBC使用接口。

用户可以通过Impala Shell或JDBC / ODBC驱动程序向Impala发起查询。 一旦查询被提交，Query Planner进程将查询请求转换为查询计划片段，然后Query Coordinator通过RPC通信，在远程impalad上执行查询。 在查询结果被发送回客户端之前，中间结果会在各个impala节点之间传输。Query Cordinator处理各个节点直接的交互，并且会聚合每个datanode产生的结果集。
Impala的查询处理过程如图3-4所示，

![](/images/20180215/impala workflow.png){:  .align-center}
**图3-4 Impala工作流**

Impalad分为Java前端与C++处理后端，接受客户端连接的Impalad即作为这次查询的Coordinator，Coordinator通过JNI调用Java前端对用户的查询SQL进行语法和语义解析并生成执行计划树，不同的操作对应不用的PlanNode, 如：SelectNode， ScanNode， SortNode， AggregationNode， HashJoinNode等等。

执行计划树的每个原子操作由一个计划片段（Plan Fragment）表示，通常一条查询语句由多个Plan Fragment组成， Plan Fragment 0表示执行树的根，汇聚结果返回给用户，执行树的叶子结点一般是Scan操作，在HDFS上可以分布式并行执行。

Java前端产生的执行计划树以Thrift数据格式返回给Impala C++后端（Coordinator）（执行计划分为多个阶段，每一个阶段叫做一个Plan Fragment，每一个Plan Fragment在执行时可以由多个Impalad实例并行执行(有些Plan Fragment只能由一个Impalad实例执行，如聚合操作)，整个执行计划为一执行计划树），由Coordinator根据执行计划，数据存储信息（Impala通过libhdfs与HDFS进行交互。通过hdfsGetHosts方法获得文件数据块所在节点的位置信息），通过调度器Coordinator::Exec把生成的执行计划树分配给相应的后端执行器Impalad执行，通过调用GetNext()方法获取计算结果，如果是insert语句，则将计算结果通过libhdfs写回HDFS当所有输入数据被消耗光，执行结束，之后注销此次查询服务。　　

##### Impala优化技术
Impala的构建充分利用了现代硬件和最新技术来实现高效的查询。 Impala使用许多工具和技术来获得最佳的查询性能。 以下讨论了Impala用于获得最佳性能的一些技巧。

###### 不使用MapReduce框架
Impala没有使用MapReduce进行并行计算。虽然MapReduce是非常好的并行计算框架，但它更多的面向批处理模式，而不是面向交互式的SQL执行。与MapReduce相比：Impala把整个查询分成一连串的执行计划树，而不是一连串的MapReduce任务，在分发执行计划后，Impala使用拉式（pull）获取数据的方式获取结果，把结果数据组成按执行树流式传递并聚合，减少的了把中间结果写入磁盘，再从磁盘读取数据的开销。在Hive中，JVM的启动开销始终是关键瓶颈，每个mapper/reducer的启动往往需要长达100ms的时间。Impala使用服务的方式避免每次执行查询都需要启动的开销，即相比Hive，Impala没了MapReduce启动时间。

###### HDFS缓存
Impala可以利用HDFS缓存从而更加有效地使用内存，特别是对于热点数据的查询。通过HDFS缓存，可以指定一个常用数据的子集来固定存储在内存中。 这适用于经常访问的表或表分区，并且足够小以适应HDFS内存缓存。
一旦设置了使用HDFS缓存，在Impala DDL中，CREATE和ALTER语句将指定缓存池的名称，以便为该表启用HDFS缓存。 实际的语法如下：
```sql
CREATE TABLE ... CACHED IN <pool name>
```
或
```sql
ALTER TABLE ... SET CACHED IN <pool name>
```

对于已经被缓存的表，如果通过ALTER TABLE ... ADD PARTITION语句添加了新分区，则这些新分区中的数据将自动缓存在同一个缓存池中。

###### 文件格式的选择
不同的文件格式和压缩编解码器适用于不同的数据集。 无论什么文件格式，Impala都可为其提升性能。 更好的数据格式允许用户在查询时利用更低的存储和优化，处理更少的数据，以及在I / O和网络期间，读取和传输更少的数据。
TEXT格式数据对于存储和查询来说效率不高，而Parquet[7]和ORC[8]格式的数据是高度优化的文件格式，可以实现更好的存储和I / O效率。 因此，将TEXT数据转换为Parquet格式可以使Impala有最好的性能。 如果数据可用作TEXT文件，则使用Parquet格式创建一个新表，并使用Impala来查询该Parquet格式的数据。 从本质上讲，Impala的设计下使用Parquet格式的效果最好。 Parquet内置了很多优化手段，这使得它适合以低延迟查询大型数据集。

###### 代码生成
Impala通过利用现代CPU的最新技术（硬件指令SSE4.2[9]），使用代码生成来优化CPU使用率和降低查询时延。代码生成优化技术具体体现在以下几个方面：
- 虚函数调用：SQL查询中的任何表达式都会导致对引擎处理的每一行数据进行一次又一次的验证。 即使表达式本身很容易计算，底层实现也会导致虚拟函数调用来验证表达式。 这会带来上下文切换昂贵的CPU开销（将当前空间保存在堆栈中，并调用虚函数）。 Impala使用内联（inline）的方式减少函数调用的开销，加快执行效率。
- LLVM：低级虚拟机（Low Level Virtual Machine）可加快查询速度并消除之前提及的开销。 我们可以将LLVM视为JVM字节码生成器。 LLVM使用某些编译器标志为编写的程序生成优化的代码。 这些类型的优化代码生成通常是在C / C ++编写的代码基础上，根据部署它们的硬件平台完成的。 唯一的区别是在SQL-on-Hadoop的case下，LLVM需要为SQL查询生成优化的代码。 LLVM包含一组库，利用这些库，我们可以为SQL查询语法树生成优化的低级语言代码。 LLVM为代码生成提供了API。Impala使用该API优化代码的生成。使用LLVM，Impala将查询速度提高了三到五倍。

##### Impala小结
Impala通过多种优化手段降低了CPU负载，比如LLVM的使用和最新的指令集的使用，这增加了I/O带宽，相比于Hive具有更好的性能。对于复杂的查询，Hive有一个多级MR流水线，对磁盘的读写有多个阶段，导致查询速度变慢。 Impala通过具有完全不同的引擎来避免这种情况，这种引擎不依赖于MR，而是在节点之间传输数据，从而有效利用集群内网络带宽。另外，Impala支持多种存储格式，如Parquet, Text, Avro, RCFile, SequeenceFile，通过选择合适的数据存储格式可以得到最好的性能。
但是Impala也存在一些不足。第一，Impala对SQL的支持不如Hive，Impala使用与Hive相同的metastore。Hive中设计TRANSFORM、JSON、XML和SerDe的任务在Impala中并不支持。第二，Impala不支持用定义函数UDF。第三，Impala不支持查询期间的容错。如果在执行过程中发生故障，则直接返回错误。Impala定位于交互式查询，一次查询失败，重新发起一次查询即可，因为每次查询的代价都很低。

#### Vertica
Vertica[10]是一个非常成熟的商用分析性数据库，它主要基于C-Store[11]和MonetDB[12]。Vertica使用分布式压缩列式架构设计对大规模分析进行了优化，使现代分析工作负载的速度更快。 Vertica是一个MPP平台，使用商品服务器并使用share nothing架构分配其工作负载。 它既是一个查询引擎也是一个存储引擎。 技术实现上，Vertica属于第四类，即使用已有的分析数据库（部署在与Hadoop不同的集群上）与Hadoop集群的datanode交互。
Vertica的架构旨在通过降低I / O来减少延迟，只使用高度压缩的列格式，只读取必要的数据。 它提供了高可扩展性，并且没有单点故障。 Vertica SQL完全符合ANSI SQL 99标准。

##### Vertica with Hadoop
Hadoop最适合于在结构化和非结构化数据集上执行大规模的ETL工作负载的任务。 对于低延迟的复杂SQL查询，Vertica在结构化数据上能提供很好的性能。 借助Vertica的Hadoop connector，分析师可以使用熟悉的BI /分析工具生成SQL代码，以便与使用Vertica的任何Hadoop分发工具进行交互。

Vertica既可以从自身的存储读取数据也可使用connector从HDFS查询数据。有了Hadoop connector，既可以在Hadoop任务中使用Vertica中存储的数据，也可以将Hadoop任务的结果存储在Vertica中。Vertica可以直接读取存储在HDFS上ORC、Parquet和Avro格式的数据。

Vertica安装部署在与Hadoop分离的集群上（如图3-5所示）。主要原因是Hadoop主要用于ETL和数据存储，而Vertica一般用于低延时的分析查询。如果Vertica安装在Hadoop集群中节点中，由于Vertica无法得知datanode具体的负载情况，也就很难满足Vertica的服务水平协议（SLA）。

![](/images/20180215/vertica architecture.jpg){:  .align-center}
**图3-5 Vertica架构**

图3-6显示了Vertica是如何通过Hadoop connector访问HDFS数据。

![](/images/20180215/vertica hdfs.png){:  .align-center}
**图3-6 Vertica访问HDFS**

Vertica的Hadoop connector包含HDFS connector和MR connector。Vertica的HDFS connector是允许Vertica在HDFS上创建表或者查询HDFS上数据而不是把数据导入到vertica中再进行查询。HDFS connector对于任何格式的数据都有一个相应的解析器。HDFS connector在Hadoop集群中的每个节点中都部署了一个实例。以下是使用HDFS connector的一个SQL实例。
```sql
CREATE EXTERNAL TABLE HadoopFile(ID VARCHAR(10), Col1 INTEGER, Col2 INTEGER,Col3 INTEGER) AS COPY SOURCE Hdfs(url='http://hadoopNameNode:50070/webhdfs/user/User1/data/input/*',username='User1’);
SELECT * FROM HadoopFile;
```

Vertica的MR connector用于创建读写Vertica数据的Hadoop MapReduce任务。在如下几个例子中会用到MR connector：
- 当一个MapReduce任务需要读取存储在Vertica中的数据时；
- 当MapReduce任务直接将数据插入到Vertica中，使用Vertica的out-of-the-box SQL功能进行实时分析时。如果数据不存在，MR connector可以为数据创建一个新表；
- 允许Pig脚本访问Vertica中的数据；

如果一个MapReduce任务要访问Vertica中的数据，MR任务会执行一个query来选出它需要的数据。该query会传给Hadoop的MapReduce API，VerticaInputFormat类的setInput方法。MR connector会把该query发送到Hadoop节点，然后Hadoop节点单独与Vertica节点建立连接，执行query并获取MR任务需要的输入数据。query有以下两种形式：
```java
VerticaInputFormat.setInput(job, "SELECT * FROM VerticaTable;");
```
或
```java
VerticaInputFormat.setInput(job, "SELECT * FROM V WHERE ID = ?", "A", "B");
```

#### MPP架构不足
下面重点看下 MPP 架构的缺点，MPP 架构最主要的缺点是不支持细粒度的容错，集群节点数量很难扩展到 100 个以上，如果集群出现落后节点，那么将影响整个系统的查询性能，此外不管 MPP 节点数量的多少，并发查询的数量通常只能达到 20 个左右。

- 容错：MPP 架构的容错特点是粗粒度容错，不能处理落后节点 (Straggler node)。粗粒度容错是指，某个 task 执行失败将导致整个查询失败，然后系统重新提交整个查询来获取结果。这种容错方式只适用于 Iterative SQL 这种低延迟的工作负载，而不适合 Batch SQL 场景，因为 Batch SQL 查询时间通常在分钟小时级别，重新提价一个查询代价太高。

- 扩展性：MPP架构要求各节点之间是对等的网络，受落后节点的影响，MPP 架构很难扩展到 100 个节点以上。如果某个节点慢于其他节点，那么整个系统的查询性能将受限于这个最慢的节点，而与集群节点数量无关。
- 并发：MPP 架构的并发查询数量和集群节点数量无关。MPP 是对称结构，当执行一个查询时，该查询将被调度到集群中的每一个节点执行，这意味着一个包含 4 个节点的 MPP 集群和一个包含 400 个节点的 MPP 集群所支持的并发查询数量是相同的，也就是说，并发查询数量和集群节点数量无关，一般而言，当并发查询个数达到 20 左右时，整个系统的吞吐已经达到满负荷状态。

### 操作型 SQL
提出Hadoop的一个主要目的就是为了对大数据集进行批处理分析。其主要组件HDFS是一个一次写入多次读取（WORM）的文件系统，Hadoop本身不是为操作型的应用设计的，对更新，删除等负载并不友好。然而实际上，一个平台的成功往往会带来更多的需求。业界对如何在Hadoop上执行事务型负载进行了探索。本节将介绍一个解决这个问题的新的工具和系统——Apache Phoenix with HBase[13]。事务型的工作负载对企业来说一直是mission-critical的任务。处理操作型工作负载在响应时间，事务保证，数据完整性，并发性和可用性方面具有非常严格的要求。

#### Apache Phoenix with HBase
Apache Phoenix是构建在HBase[14]之上的一个SQL引擎，并使用内置的JDBC驱动对HBase中的数据进行低延迟访问。Phoenix在Hadoop生态中的位置如图3-7所示。HBase是一个NoSQL的分布式数据库，其利用HDFS作为底层存储，可存储、操作大量的非关系型数据。 Phoenix可以将HBase数据模型映射到关系模型，并且可将用户编写的SQL查询编译为一系列的scan操作。最终产生通用的JDBC结果集返回给客户端。 Phoenix可以用于在HBase之上为使用SQL的操作型工作负载构建事务处理系统。 Phoenix有着很好的SQL支持和覆盖和性能，小范围的查询时延在毫秒级，千万级数据的查询时延在秒级。Phoenix从4.4.0版本开始支持用户定义函数（UDF）。
![](/images/20180215/phoenix hbase.jpg){:  .align-center}
**图3-7 Phoenix在Hadoop生态中的位置**

Apache Phoenix的主要组件包括查询引擎，元数据存储，JDBC驱动。它支持HBase的组合键，并在HBase面向字节的数据上提供SQL数据类型。Phoenix也提供HBase数据上的只读视图，支持skema动态扩展。Phoenix还支持所有的SQL连接语法和相关子查询。事务支持是在OLTP工作负载中是非常重要的。 Phoenix系统内部使用Tephra[15]的开源框架为事务性语义提供支持。 Phoenix支持快照隔离[16]，事务可以看到自己的未提交的数据，并使用乐观并发控制协议[17]进行并发控制。HBase数据很难通过自己的命令行界面（CLI）以及专有的脚本和Java API来访问。 Phoenix使用SQL访问HBase数据非常方便，并且在HBase架构上做了许多优化工作，以优化SQL查询并降低延迟。

Phoenix可以将SQL查询编译为低级HBase API调用（put，delete，scan等）。 Phoenix是位于HBase Region Server上的一个轻量级的进程。 它并行执行HBase行scan。 图3-8显示了Phoenix几个重要组件。 Phoenix JDBC驱动程序作为jar包安装在客户端，Phoenix可执行jar包安装在HBase的Region Server上。

![](/images/20180215/phoenix architecture.jpg){:  .align-center}
**图3-8 Phoenix架构**

Phoenix客户端是Phoenix查询服务器的一部分， 它包含HBase的JDBC和ODBC驱动程序。 Phoenix查询引擎是基于SQL的查询引擎的典型代表，包含解析器，查询计划器和优化器。与新版本的Hive一样，Phoenix使用Apache Calcite[18]作为查询优化器。 Phoenix的查询执行引擎尽可能多地将计算推向HBase server，利用HBase API。Phoenix查询引擎的功能模块图如图3-9所示。

![](/images/20180215/phoenix query.jpg){:  .align-center}
**图3-9 Phoenix查询引擎**

Phoenix系统内部使用Tephra的开源框架支持事务。图3-10显示了Phoenix中的事务管理器的总体架构。Tephra在HBase之上提供了事务的全局一致性。Tephra在内部使用HBase对多版本控制的原生支持，也称为多版本并发控制（简称MVCC或MCC），用于事务读写。 每个事务都会看到它自己的数据“快照”，提供即时的并发事务的快照隔离。

![](/images/20180215/phoenix tx.jpg){:  .align-center}
**图3-10 Phoenix事务管理器**

Phoenix内部使用上述架构来支持HBase上的SQL事务。默认配置下，Phoenix支持可重复读的隔离级别。

#### Phoenix中的优化
Phoenix内部做了一些优化来降低查询时延。具体有以下四点。
1. 支持二级索引。在HBase中，只有一个单一的按照字典序排序的rowKey索引，当使用rowKey来进行数据查询的时候速度较快，但是如果不使用rowKey来查询的话就会使用filter来对全表进行扫描，很大程度上降低了检索性能。而Phoenix提供了二级索引技术来应对这种使用rowKey之外的条件进行检索的场景。
- Covered Indexes。只需要通过索引就能返回所要查询的数据，所以索引的列必须包含所需查询的列(SELECT的列和WHRER的列)
- Functional Indexes。从Phoeinx4.3以上就支持函数索引，其索引不局限于列，可以合适任意的表达式来创建索引，当在查询时用到了这些表达式时就直接返回表达式结果。
- Global Indexes。Global indexing适用于多读少写的业务场景。使用Global indexing的话在写数据的时候会消耗大量开销，因为所有对数据表的更新操作（DELETE, UPSERT VALUES and UPSERT SELECT）,会引起索引表的更新，而索引表是分布在不同的数据节点上的，跨节点的数据传输带来了较大的性能消耗。在读数据的时候Phoenix会选择索引表来降低查询消耗的时间。在默认情况下如果想查询的字段不是索引字段的话索引表不会被使用，也就是说不会带来查询速度的提升。
- Local Indexes。Local indexing适用于写操作频繁的场景。与Global indexing一样，Phoenix会自动判定在进行查询的时候是否使用索引。使用Local indexing时，索引数据和数据表的数据是存放在相同的服务器中的避免了在写操作的时候往不同服务器的索引表中写索引带来的额外开销。使用Local indexing的时候即使查询的字段不是索引字段索引表也会被使用，这会带来查询速度的提升，这点跟Global indexing不同。一个数据表的所有索引数据都存储在一个单一的独立的可共享的表中。
2. 时间戳。从4.6版本开始，Phoenix提供了一种将HBase原生的row timestamp映射到Phoenix列的方法。这样有利于充分利用HBase提供的针对存储文件的时间范围的各种优化，以及Phoenix内置的各种查询优化。
3. 分页查询
4. 散步表。如果row key是自动增长的，那么HBase的顺序写会导致region server产生数据热点的问题，Phoenix的Salted Tables技术可以解决region server的热点问题。


## 性能优化
本小节主要介绍这些SQL-on-Hadoop解决方案做的性能优化的工作。主要介绍磁盘IO和CPU使用率两个方面的优化。

### 提高数据访问局部性
SQL-on-Hadoop系统设计的一个基本的原则就是将计算推到数据而不是将数据推到计算。因为数据分布在不同的节点上，如果过于频发地在节点直接移动数据将会产生大量的网络传输开销。调度系统在进行任务调度时，应该尽可能保证计算任务正好分配到数据所在的节点上，此时无需任何的数据传输，如若无法保证节点局部性，那么应该尽可能保证计算任务分配到数据所在的机架上，因为我们认为机架内部的网络传输速度远高于跨机架的网络传输速度。

为了提高数据访问局部性，调度系统会结合延迟调度算法来进行任务调度。该算法的核心思想是优先将计算任务调度到数据所在的节点 i，如果节点 i 没有足够的计算资源，等待几秒钟后，如果节点 i 依然没有计算资源可用，那么就将该计算任务调度到其他计算节点。

### 列式存储
传统的关系存储模型将一个元组的列连续存储（行存），即使只查询一个列，也需要将整个元组读取出来。可以发现，当查询只有少量列时，性能非常低。

列存储的思想是将元组垂直切分为列族集合，每一个列族独立存储，列族可以退化为只仅包含一个列的平凡列族。当查询少量列时，列存储模型可以极大的减少磁盘 IO 操作，提高查询性能。当查询的列跨越多个列族时，需要将存储在不同列族中列数据拼接成原始数据，由于不同列族存储在不同的 HDFS 节点上，导致大量的数据跨越网络传输，从而降低查询性能。因此在实际使用列族时，通常根据业务查询特点，将频繁访问的列放在一个列族中。

### 分区
分区就是指按照特定的分区策略将一张表水平或者垂直切分为多个子表，不同的子表存储在不同的节点上以提高数据处理的并行度。比较典型的分区策略又哈希，Range等。

SQL-on-Hadoop中也存在表分区的机制，一个表分区存储在一个HDFS目录下。以Hive为例，当我们执行以下SQL：
```sql
CREATE TABLE table1(s_id string,s_name string,s_date string) PARTITION BY(s_date string)
```

当我们向table1中插入如下元组时：
```
(s_id=‘10001’,s_name=‘tom’,s_date=‘1995-06-06’)
(s_id=‘10002’,s_name=‘jane’,s_date=‘1995-04-10’)
```

HDFS将创建如下目录：
```
/user/hive/warehouse/table1/s_date=1995-06-06
/user/hive/warehouse/table1/s_date=1995-04-10
```

当执行 ```SELECT * FROM table1 WHERE s_date=’1995-06-06’```时，只需要扫描 s_date=1995-06-06目录即可，这样可以跳过大量无关数据的扫描，从而加快数据查询速度。

目前大部分 SQL On Hadoop 都支持分区功能，比如 Hive，Presto，Impala。

### 基于开销的查询优化器（CBO）
目前 SQL On Hadoop 的查询优化器主要有两种：基于规则的 (Rule-Based Optimizer) 和基于代价的 (Cost-Based Optimizer CBO)。基于规则的优化器简单，易于实现，通过内置的一组规则来决定如何执行查询计划，这里不做介绍。

设计一个好的 CBO 优化器非常具有挑战性，一个好的 CBO 依赖于详细可靠的统计信息，比如每个列的最大值，最小值，表大小，表分区信息，桶信息，然而在 SQL On Hadoop 中，通常缺乏可靠的统计结果，代价估计代数，这使得在 SQL On Hadoop 中引入 CBO 很困难。尽管如此，鉴于 CBO 在运行可以更加智能的进行查询优化，仍然有越来越多的 SQL On Hadoop 开始支持 CBO，比如 Hive，从0.13版本开始，Hive使用了开源的企业级的查询优化器Calcite。。

### 向量化查询执行引擎
查询执行引擎 (query execution engine) 是 SQL On Hadoop 的核心组件。查询执行引擎的好坏对查询性能的影响非常大。目前主要有两种查询执行：火山模型和向量化执行引擎。
我们知道火山模型（Volcano Model）是最早的查询执行引擎。在这种模型中，查询计划是一个由 操作 组成的 树，其中每一个 operator 包含三个函数：open，next，close。open 用于申请资源，比如分配内存，打开文件，close 用于释放资源，next 方法递归的调用子 operator 的 next 方法取出一个元组。图4-1显示了一个join算子的火山模型。

![](/images/20180215/volcano model.png){:  .align-center}
**图4-1 火山模型**

火山模型的主要缺点是低下的 CPU Cache 命中率。next 方法一次只返回一个元组，元组通常采用行存储，如图4-2 Row Format，如果顺序访问第一列 1，2，3，那么每次访问都将导致 CPU Cache 命中失败 (假设该行不能完全放入 CPU Cache 中)。如果采用 Column Format，那么只有在访问第一个值时才出现缓存命中失败，后续访问 2 和 3 时都将缓存命中成功, 从而极大的提高查询性能。

![](/images/20180215/row column.png){:  .align-center}
**图4-2 行存 vs 列存**

而向量化执行引擎的next调用不是返回一个完整的列，而是返回一个可以放到CPU Cache中的向量。这大大降低了扫描表，聚合函数和连接表操作的CPU使用率。近几年，一些 SQL On Hadoop 系统引入了向量化执行引擎，比如 Hive，Impala，Presto 等，尽管其实现细节不同，但核心思想是一致的：尽可能的在一次 next 方法调用返回多条数据，然后使用动态代码生成技术来优化循环，表达式计算从而减少解释开销，提高 CPU Cache 命中率，减少分支预测。

## 新特性
本节介绍一些SQL-on-Hadoop工作中的新特性。

### GPU is the New CPU
首先介绍的是利用GPU加速计算的新的SQL引擎。
MapD[18]是一个使用GPU驱动的数据库和可视化分析平台，能够以极低的延迟提供极大数据集的交互式查询。 MapD利用GPU在每台服务器上将近40,000个核心并行执行SQL查询，从而大大提高了其SQL引擎的速度。 它还利用图形处理功能为复杂数据集提供数据可视化。

目前，MapD只能架构在单个节点上部署，最多可配置8个Nvidia顶级的Tesla K80协处理器; 目前的最新工作是使其成为一个分布式数据库。 像Impala和Spark SQL一样，MapD也使用LLVM来动态生成和编译SQL代码，以实现大规模的加速。 它是用C ++编写的，并且利用CUDA和OpenCL与GPU进行通信。

MapD不是一个分布式的SQL引擎。 目前，它被设计成在独立的机器上工作。 MapD的一个开源竞争对手是GPUdb[19]，它是一个具有SQL功能的分布式数据库。 GPUdb为开发人员和业务分析人员提供ODBC连接器和API，以便与数据库进行交互并提供标准的SQL接口。 它提供ANSI SQL 92兼容的SQL接口，并且还具有RESTful API。
GPUdb目前支持NVIDIA GPU和Xeon Phi多核设备。 它很像MPP引擎，其中核心引擎部署在集群中的每个节点上，最好是具有相同的节点和相同数量的GPU。 选择一个节点作为协调节点。 集群可以扩展或者缩小，具体取决于存储和查询效率要求。

### HTAP

HTAP代表Hybrid Transactional/Analytical Processing。 在传统的企业中，有一套数据库处理系统可以使运行中的应用程序保持在企业的功能上。 这些数据库处理系统大多是事务处理系统，始终应该处于开启状态，并且可以实时处理数据。另一方面是一套用于分析，BI，报告和数据仓库和批处理（如ETL）的数据库处理系统。通常来说AP和TP是两套系统，但是随着信息时代的发展，如见的企业既需要即时分析，也需要数据驱动的决策。 通过分层架构提供分析的旧方法已经不够用了。于是就产生了HTAP这个想法。
HTAP的思想是提供一个统一的系统，最终用户无需使用不同的系统来应对不同的负载。 这意味着需要创建一个结合操作型和分析型的环境HTAP。创造一个HTAP系统的难点在于：
- 一个查询引擎处理所有类型的负载；
- 支持多种存储引擎；
- 所有类型的负载使用同一种数据模型；
- 企业级的特性，如安全，容错，并发等；

目前在做这方面工作（NewSQL引擎）的企业不多，有VoltDB[20]，NuoDB和Trafodion。

## 结论

本文讨论了SQL和Hadoop两种技术融合的方式和典型代表，并分析了各个方式的优缺点。同时本文也分析了各个系统对性能的优化，尽管目前在SQL-on-Hadoop领域中存在种类繁杂的系统与工具， 尽管实现细节和应用场景不同。最后也探索了未来SQL-on-Hadoop的发展方向。
我认为无论实现方式如何所有的 SQL On Hadoop 都应该尽可能的追求快速，易使用。查询速度越快，就越能适应更多的场景。另外，支持 ANS标准 SQL可以减少用户学习曲线，避免用户陷入到过多的语言特性中。

## 参考文献
[1] Apache Hadoop [https://hadoop.apache.org](https://hadoop.apache.org)

[2] Oracle [www.oracle.com](www.oracle.com)

[3] DB2 [https://www.ibm.com/analytics/us/en/db2/](https://www.ibm.com/analytics/us/en/db2/)

[4] Thusoo A, Sarma J S, Jain N, et al. Hive: a warehousing solution over a map-reduce framework[J]. Proceedings of the VLDB Endowment, 2009, 2(2): 1626-1629.

[5] Saha B, Shah H, Seth S, et al. Apache tez: A unifying framework for modeling and building data processing applications[C]//Proceedings of the 2015 ACM SIGMOD international conference on Management of Data. ACM, 2015: 1357-1369.

[6] Bittorf M, Bobrovytsky T, Erickson C C A C J, et al. Impala: A modern, open-source SQL engine for Hadoop[C]//Proceedings of the 7th Biennial Conference on Innovative Data Systems Research. 2015.

[7] Kestelyn J. Introducing Parquet: Efficient Columnar Storage for Apache Hadoop[J]. Cloudera Blog, 2013, 3.

[8] Apache ORC [https://orc.apache.org/](https://orc.apache.org/)

[9] SSE4.2指令 [https://en.wikipedia.org/wiki/SSE4#SSE4.2](https://en.wikipedia.org/wiki/SSE4#SSE4.2)

[10] Lamb A, Fuller M, Varadarajan R, et al. The vertica analytic database: C-store 7 years later[J]. Proceedings of the VLDB Endowment, 2012, 5(12): 1790-1801.

[11] Stonebraker M, Abadi D J, Batkin A, et al. C-store: a column-oriented DBMS[C]//Proceedings of the 31st international conference on Very large data bases. VLDB Endowment, 2005: 553-564.

[12] Idreos S, Groffen F, Nes N, et al. MonetDB: Two decades of research in column-oriented database architectures[J]. A Quarterly Bulletin of the IEEE Computer Society Technical Committee on Database Engineering, 2012, 35(1): 40-45.

[13] HBase A. A distributed database for large datasets[J]. The Apache Software Foundation, Los Angeles, CA. URL [http://hbase.apache.org](http://hbase.apache.org), 4(4.2).

[14] Apache Phoenix [https://phoenix.apache.org/](https://phoenix.apache.org/)

[15] Tephra [http://tephra.apache.org/](http://tephra.apache.org/)

[16] Berenson H, Bernstein P, Gray J, et al. A critique of ANSI SQL isolation levels[C]//ACM SIGMOD Record. ACM, 1995, 24(2): 1-10.

[17] Kung H T, Robinson J T. On optimistic methods for concurrency control[J]. ACM Transactions on Database Systems (TODS), 1981, 6(2): 213-226.

[18] MapD [https://www.mapd.com/](https://www.mapd.com/)

[19] GPUdb [https://www.kinetica.com/](https://www.kinetica.com/)

[20] Stonebraker M, Weisberg A. The VoltDB Main Memory DBMS[J]. IEEE Data Eng. Bull., 2013, 36(2): 21-27.


