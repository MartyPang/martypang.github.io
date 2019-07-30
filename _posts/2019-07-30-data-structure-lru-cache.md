---
layout: post
title: 'Implement a basic LRU cache'
author: Marty Pang
categories: 
  - Data Structure
tags: 
  - LRU Cache
  - Linked List
last_modified_at: 2019-07-30T15:52:09-05:00
---

缓存在计算机组成中占据很重要的角色，例如CPU的多级缓存结构，包括L1缓存，L2缓存，L3缓存等，他们的作用就是为了尽可能消除CPU与主存处理数据速度的差距；再比如一些内存数据库系统如Redis，尽可能把数据都存在内存中，做一层缓存，减少磁盘IO。

速度越快的存储一般价格也越贵，其容量是有限的，当其作为缓存达到容量限制的时候，就需要应用一些替换策略，把一些不怎么访问的条目替换出去以保持一个较高的缓存命中。常用的替换策略包括：

- FIFO - **F**irst **I**n **F**irst **O**ut
即先进先出策略，该原则简单公平，使用队列即可实现。
- LFU - **L**east **F**requently **U**sed
最不经常使用，如果某条目的数据在一段时间内的访问频率不高，那么将来一段时间内被使用的可能性也很小，优先淘汰。
- LRU - **L**east **R**ecently **U**sed
字面意思，最近最少使用的条目优先移除缓存。LRU的出现主要反映了数据访问的局部性原理，即刚访问过的数据有很高的概率再次被访问。

本文主要实现一个简单的LRU缓存。


* 目录
{:toc}

# 需求

缓存是用来提供快速高效的数据访问方式，它必须满足以下一些设计需求：
- 大小限制：缓存容量需要受一些界限的限制，防止内存耗尽；
- 快速访问：对缓存的插入与查找操作应该是$O(1)$的时间复杂度；
- 设计相应的替换算法：在达到缓存容量时，使用LRU替换最近最少使用的条目；

# 实现

LRU要求缓存替换最近最少使用的条目，我们需要相应的数据结构追踪最近使用的条目，包括最近使用以及最少使用。另外还需提供$O(1)$复杂度的增删查。[HashMap](https://www.hytheory.com/java/Java-Collection-II-Map/)能满足$O(1)$复杂度查询，插入与删除，然而HashMap中的条目是无顺序的，单独使用这样一个结构无法追踪条目的使用情况。故我们需要另外一个结构来追踪缓存条目的访问情况，对该结构的插入删除也应该是$O(1)$的。在LRU的情况下，我们选择双向链表结构。
一般来说，针对双向链表的插入与删除操作的时间复杂度是$O(n)$，而在拥有HashMap与使用LRU算法的前提下，插入新条目都是在双向链表头部插入，删除操作均是尾部，HashMap也使得节点定位只需要$O(1)$的复杂度。

![LRUCache](/images/20190730/lru.png){:	.align-center}

## 接口

缓存提供两个最基本的功能，即`put`与`get`。使用泛型，使得缓存接受任何类型的条目。

```java
public interface ICache<K, V> {
    public V get(K key);
    public void put(K key, V value);
}
```

## 条目

本文的实现是偷懒版本，使用Java的Deque实现，Entry重写`equals`函数。

```java
public class Entry<K, V> {
    K key;
    V value;
    public Entry(K k, V v) {
        key = k;
        value = v;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Entry<?, ?> entry = (Entry<?, ?>) o;
        return Objects.equals(key, entry.key) &&
                Objects.equals(value, entry.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(key, value);
    }
}
```

## LRUCache

一个最简单的LRUCache使用HashMap与Deque实现，包含以下三个成员变量。

```java
public class LRUCache<K, V> implements ICache<K, V> {
    /**
     * A double linked queue storing entries.
     */
    private Deque<Entry<K, V>> deque;
    /**
     * A hashmap maps keys to values accordingly,
     * making query complexity O(1).
     */
    private Map<K, Entry<K, V>> cache;
    /**
     * capacity bound indicating memory limits.
     */
    private int capacity;
}
```

定义默认构造函数与调整初始大小构造函数，假设我们把默认的缓存大小定义为8。

```java
public LRUCache() {
    deque = new LinkedList<>();
    cache = new HashMap<>();
    capacity = 8;
}

public LRUCache(int size) {
    deque = new LinkedList<>();
    cache = new HashMap<>();
    capacity = Math.max(1<<3, size);
}
```
## get & put

接下来就是实现ICache定义的两个基本的接口了。先看get，首先检查cache是否有该key对应的条目，如果没有直接返回null。如果有，则将对应的条目移到双向链表的头部，并返回相应的值。

```java
@Override
public V get(K key) {
    Entry<K, V> entry = cache.get(key);
    if(entry == null) {
        return null;
    }
    if(!entry.equals(deque.getFirst())) {
        // reset entry as the most recent used entry,
        // add it to the head of the list
        if(entry.equals(deque.getLast())) {
            deque.pollLast();
        } else {
            deque.remove(entry);
        }
        deque.addFirst(entry);
    }
    return entry.value;

}
```

put操作同样先判断cache是否已经有对应的条目，如果存在则直接更新相应的值，并讲该条目移动到链表头部。对于不存在key对应条目的情况，如果cache还有空间，则直接把新生成的条目插入链表头部即可。否则移除链表尾部的条目，再将新条目插入头部。

```java
@Override
public void put(K key, V value) {
    if(cache.containsKey(key)) {
        Entry<K, V> entry = cache.get(key);
        entry.value = value;
        // this entry being the most recent used one,
        // set it to the head of the deque
        if(entry.equals(deque.getLast())) {
            deque.pollLast();
        } else {
            deque.remove(entry);
        }
        deque.addFirst(entry);
        return;
    }
    Entry<K, V> newEntry = new Entry<>(key, value);
    // Still got room for new entry
    if(cache.size() >= capacity) {
        //Remove the least recent used entry, a.k.a, the tail node
        Entry<K, V> leastrecentused = deque.pollLast();
        cache.remove(leastrecentused);
    }
    deque.addFirst(newEntry);
    cache.put(key, newEntry);
}
```

## 测试

至此，一个简单的LRUCache就实现了，注意由于使用了HashMap，所以该LRUCache是非线程安全的。

我们可以写个简单的main函数测试一下。

```java
public class Main {
    public static void main(String[] args) {
        LRUCache<String, Integer> cache = new LRUCache<>(8);
        cache.put("a", 5);
        cache.put("b", 1);
        cache.put("c", 7);
        cache.put("d", 10);
        cache.put("a", 5);
        cache.put("f", 1);
        cache.put("r", 7);
        cache.put("b", 10);
        cache.put("q", 5);
        cache.put("t", 1);
        cache.put("r", 7);
        cache.put("y", 10);
    }
}
```
输出应当如下：

```
(a,5)
(b,1)(a,5)
(c,7)(b,1)(a,5)
(d,10)(c,7)(b,1)(a,5)
(a,5)(d,10)(c,7)(b,1)
(f,1)(a,5)(d,10)(c,7)(b,1)
(r,7)(f,1)(a,5)(d,10)(c,7)(b,1)
(b,10)(r,7)(f,1)(a,5)(d,10)(c,7)
(q,5)(b,10)(r,7)(f,1)(a,5)(d,10)(c,7)
(t,1)(q,5)(b,10)(r,7)(f,1)(a,5)(d,10)(c,7)
(r,7)(t,1)(q,5)(b,10)(f,1)(a,5)(d,10)(c,7)
(y,10)(r,7)(t,1)(q,5)(b,10)(f,1)(a,5)(d,10)
```

完整的代码参考[GitHub](https://github.com/MartyPang/DataStructures/tree/master/src/main/java/ecnu/dase/cache)。