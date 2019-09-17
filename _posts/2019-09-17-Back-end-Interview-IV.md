---
layout: post
title: 'Java后端面试整理（三）'
author: Marty Pang
categories: 
  - Interview
tags: 
  - Java
  - Back-End
last_modified_at: 2019-08-30T12:52:09-05:00
---

前三篇面试题总结主要针对Java基础、操作系统与网络、数据库，后端面试经常还会问到一些分布式框架，例如kafka、zookeeper和dubbo等。

* 目录
{:toc}


# 消息队列

1. 消息队列的主要作用？有哪些常用的消息队列？
2. 为什么要使用消息队列？有哪些应用场景？
3. 消息队列有哪些优缺点？
4. 消息队列如何保证高可用？
5. 如何保证消息的顺序性？
6. 如何保证消息不被重复消费？（幂等性问题）Kafka中是如何保证的？
7. 如何保证消息的可靠传输？如何处理消息丢失的问题？
8. 如果让你写一个消息队列，该如何进行架构设计啊？说一下你的思路
9. 如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？
10. kafka中的 zookeeper 起到什么作用，可以不用zookeeper么？  
zookeeper 是一个分布式的协调组件，早期版本的kafka用zk做meta信息存储，consumer的消费状态，group的管理以及 offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题，新版本中逐渐弱化了zookeeper的作用。新的consumer使用了kafka内部的group coordination协议，也减少了对zookeeper的依赖。但是broker依然依赖于ZK，zookeeper 在kafka中还用来选举controller 和 检测broker是否存活等等。
