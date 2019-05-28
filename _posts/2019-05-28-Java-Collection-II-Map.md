---
layout: post
title: 'Java Collection I - General Framework'
author: Marty Pang
categories: 
  - Java
tags: 
  - Collection
last_modified_at: 2019-05-07T14:25:09-05:00
---

本文介绍Java中的Map接口以及对应的实现类。[Java Collection系列第一篇](https://www.hytheory.com/java/Java-Collection-I-General-Framework/)整理了集合框架和Map的框架，下面两张图简单回顾一下。

![CollectionModel](/images/20190507/CollectionModel.png){:	.align-center}
<center>Collection总体框架图<center>

Java集合主要有List集合，Set集合，Map映射以及工具类（Arrays，Collections，Iterator），上图是继承Collection接口的类。Map不继承Collection，框架图如下。

![MapModel](/images/20190507/MapModel.png){:	.align-center}
<center>Map总体框架图</center>


