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

Java的Collection框架是对集合的高度抽象，围绕一组标准接口而设计，其包含一些列的接口以及对应的实现，并且提供几个工具类实现包括排序功能等算法。正确应用Collection框架能够大大方便编程人员的开发。本文以及后序的Java Collection系列文章会详细分析每个接口与类。

![CollectionModel](/images/20190507/CollectionModel.png){:	.align-center}
<center>Collection总体框架图<center>

Java集合主要有List集合，Set集合，Map映射以及工具类（Arrays，Collections，Iterator），上图是继承Collection接口的类。Map不继承Collection，框架图如下。

![MapModel](/images/20190507/MapModel.png){:	.align-center}
<center>Map总体框架图</center>

## Iterable

JDK 1.5之前的Collection接口是继承Iterator迭代器接口的，迭代器接口提供了遍历集合元素的功能，有以下三个接口。

- `hasNext()`，如果迭代器有更多元素，则返回true，否则返回false；
- `next()`，返回下一个元素，若无元素，则抛出`NoSuchElementException`异常；
- `remove()`，删除迭代器返回的最后一个元素；

1.5版本以后，Collection接口继承Iterable接口，很多博客的框架图仍旧是继承Iterator接口。Iterable接口包含`iterator()`返回迭代器，以及一个`forEach()`接口，为增强的for语句服务。下面是一个简单的增强for语句的例子。

```java
List<Person> persons = new ArrayList<>();
persons.add(p1);
persons.add(p2);
persons.add(p3);
for(Person p : persons) {
    p.eat();
}
```

## Collection

Collection是一个包含集合的基本属性与操作的高度抽象的接口，有List和Set两个分支（Queue最终由LinkedList实现）。

- List是一组元素的有序集合，实现类包括常用的ArrayList，Vector，LinkedList等。
- Set是一组不重复元素的集合，可以是无序的，比如HashSet，也可以是有序的，如实现类TreeSet。事实上，通过源码可以知道，HashSet的实现是依赖于HashMap的，同样TreeSet依赖于TreeMap。

Collection接口中声明了许多方法，下面列出常用的一些接口。

|                   方法                    |              描述              |
| :---------------------------------------: | :----------------------------: |
|                int size()                 |      返回集合中元素的个数      |
|             boolean add(E e)              |      往集合中插入指定元素      |
| boolean addAll(Collection\<? extends E\> c) | 在调用集合中插入指定的集合元素 |
|         boolean remove(Object o)          |          删除指定元素          |
|        boolean contains(Object o)         |    判断集合是否包含某个元素    |
|               void clear()                |       删除集合的所有元素       |
|         boolean equals(Object o)          |      比较两个集合是否相等      |


## Map

Map是映射接口，并没有继承Collection接口，集合中的每个元素均是一个key-value对。AbstractMap实现了大部分的Map接口。HashMap、TreeMap与WeakHashMap均继承了AbstractMap，且不是线程同步的。下表罗列了一些常用的Map接口。

|                    方法                    |            描述             |
| :----------------------------------------: | :-------------------------: |
|           V put(K key, V value)            |     往map中插入一个kv对     |
|       V putIfAbsent(K key, V value)        |   若map不存在key，则插入    |
|            V remove(Object key)            |      删除指定key的kv对      |
|                Set keySet()                |    返回key集合，用于遍历    |
|       Set<Map.Entry<K,V>> entrySet()       |      返回key-value集合      |
|             V get(Object key)              |     返回key对应的value      |
| V getOrDefault(Object key, V defaultValue) | 若key不存在则返回一个默认值 |

以上就是Java Collection框架抽象层的一个概述，之后的文章逐个分析实现类以及工具类。:relaxed:
