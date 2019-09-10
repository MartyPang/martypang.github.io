---
layout: post
title: 'Java后端面试整理（一）'
author: Marty Pang
categories: 
  - Interview
tags: 
  - Java
  - Back-End
last_modified_at: 2019-08-29T12:52:09-05:00
---

磨人的秋招，再也不想经历第二次(′д｀ )…彡…彡...在此记录一下准备秋招面试过程中整理的关于后端开发的一些基础知识，奈何博主从未使用过一些后端框架、缓存以及消息队列甚至一些分布式服务框架（那你为啥还找后端开发？？？），整理的东西大多数都是一些基础。

* 目录
{:toc}

# Java语言基础

1. String类能被继承吗，为什么？
  不能，String类被final修饰。

2. String、StringBuffer、StringBuilder的区别？对应的使用场景？
  String是不可变的字符串常量，final修饰，StringBuffer和StringBuilder是字符缓冲变量，内容可变。StringBuffer和StringBuilder功能一致，可以append，insert，delete等，前者使用synchronized支持线程安全。StringBuilder非线程安全，速度最快。
  在字符串不经常变化的场景中可以使用String类；频繁字符串运算，并且多线程环境中，使用StringBuffer；StringBuilder则单线程环境下。

3. 如何实现不可变的类？String、Long、Double都是不可变类
  final修饰类，所有成员是私有，不可变成员用final修饰，只能在构造的时候赋值一次，没有setter，getter返回对象的拷贝。

4. final的作用？
- final修饰类，类不能被继承，成员方法隐性设置为final修饰
- final修饰方法，方法不能被重写，但可以被继承，private方法隐式为final修饰
- final修饰类成员变量，只能在初始化活在构造器中赋值一次，修饰引用类型时，指针不变，内容可变

5. 讲讲类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，当new的时候，他们的执行顺序？
  父类的静态代码块 -> 子类静态代码块 -> 父类普通代码块 -> 父类构造函数 -> 子类普通代码块 -> 子类构造函数 。

6. Java中new一个对象发生了什么？
- 加载类到内存中（方法区）；
- 虚拟机栈分配引用；
- 堆中开辟内存，创建对象；
- 初始化数据，填充对象头，成员变量初始化默认的值；
- 引用指向堆中的对象；

7. JAVA8的ConcurrentHashMap为什么放弃了分段锁，有什么问题吗
  多个分段锁浪费内存；map在put时竞争同一个锁的概率小，分段锁造成更新等操作时间长，1.8的CAS操作效率更高，如果没有竞争直接更新即可，CAS失败再使用synchronized。

8. LinkedHashMap是如何保证顺序的？
  使用一个双向链表维护条目的访问顺序，可用作LRU的实现。

9. 在自己的代码中，如果创建一个java.lang.String类，这个类是否可以被类加载器加载？为什么。
  不能，双亲委派模式，使用默认的应用类加载器时，会将加载请求向父加载器转发，顶层引导类加载器搜索JAVA_HOME/lib，查找到自带的String类，直接加载。

10. Exception与error的差别   Exception的分类？
- Exception和Error都继承与Throwable，是Java异常处理机制的基本组成类型；
- Exception是程序正常运行中，可预料的意外情况，可能并且应该被捕获并进行相应的处理；
- Error指在正常情况下不大可能出现的情况，Error一般会导致程序处理非正常，不可恢复状态，比如OOM。
Exception分为运行时异常与非运行时异常，前者包括除0，数组越界等；后者

11. 以下代码的输出结果是？
```java
Integer a = 11;
Integer b = 11;
Integer c = 157;
Integer d = 157;
System.out.println((a == b) + " " + (c == d));
```
输出`true false`，	Java 基础数据类型包装类的高频区间的数据缓存如下：
- Byte：`[-128, 127]`
- Short：`[-128, 127]`
- Character：`[0, 127]`
- Long：`[-128, 127]`
- Integer：`[-128, 127]`
如果包装类的值在这个区间内，直接复用已有对象。

## equals & hashcode

1. equals 和 ==
   ==比较两个变量的值，如果两个变量是基本类型，直接比较值。如果是引用类型，则比较对象的引用，即地址。Object中的equals方法就是用==实现的，如果要比较堆中的两个对象是否相同，则要重写equals方法，比较对象的内容。

2. equals 和 hashcode
   equals返回true的两个对象hashcode一定相等。

## Collections

1. ArrayList、LinkedList、Stack底层？插入删除效率？
  ArrayList底层Object数组，查找效率高，一个偏移量即可，增删效率低，vector为线程安全版本，为了优化性能，copyonwritearraylist；
  LinkedList底层基于双向链表实现，查找效率低，增删$O(1)$；
  Stack继承Vector，底层为数组实现；
  
## HashMap、HashTable、ConcurrentHashMap

1. 简单介绍一下HashMap、HashTable、ConcurrentHashMap。
HashTable底层为数组+链表的实现，key和value均不允许为null，使用synchronized关键字实现线程安全，锁住整个哈希表。初始大小为11，扩容2倍加1。
HashMap底层为数组+链表+红黑树实现，线程不安全，达到负载因子\*容量大小之后会扩容，两倍原始容量，链表长度大于8之后，转为红黑树存储。
ConcurrentHashMap在JDK1.7数组分段加锁（可重入锁），读操作不加锁，entry的key和value是volatile修饰的。JDK1.8取消了分段，使用CAS操作，CAS操作失败使用synchronized。

2. JDK1.7与JDK1.8的HashMap实现有什么区别？
JDK1.8之后，当链表大于8后，使用红黑树，减少搜索时间。

3. HashMap的put与get的过程是怎样的？
   get首先将key hash以后获得相应的桶，如果桶为null，则直接返回null。判断桶的第一个位置，如果key为查询的key则直接返回node；如果第一个节点不匹配，则根据节点类型（红黑树 or 链表），获取key对应的node。
   put同样判断桶数组是否为空，若为空则进行初始化。再获取hash对应的桶，若桶为空则表明没有hash冲突，直接初始化一个node即可。若不为空，则比较第一个节点的key与put的key是否相同，相同表明该节点已经存在，直接更新值。否则根据第一个节点的类型（红黑树/链表）决定调用什么方法进行插入，若为链表，遍历链表，若不存在key对应的节点，则在尾部插入新节点。成功put之后判断是否需要resize。

4. ConcurrentHashMap的put与get流程。
   get基本与HashMap一个流程；
   put首先判断桶数组是否为空，是否需要初始化。若不为空，则根据哈希值获取相应桶的第一个节点，如果第一个节点为空，直接通过原子操作插入即可，无需锁。如果发现第一个节点哈希为-1，说明map正在resize。如果以上情况均不满足，那么就需要使用synchronized对第一个节点上锁。
   
5. HashMap扩容的时机以及具体的做法？
当`hashmap.size >= capacity * loadfactor`时，HashMap进行resize操作。具体步骤如下：
  - 扩容，new一个新的entry数组，长度是原数组的两倍；
  - 1.7中rehash，遍历原entry数组，把所有entry重新hash到新数组，用的是头插法，而1.8中，并没有重新rehash，而是原始位置加原数组长度得到新位置，是否要移动由`e.hash & oldCap`的最高位决定，使用尾插法。

6. HashMap的大小为什么要2的幂次？
计算新位置的时候`e.hash & (capacity -1)`，其中`capacity-1`的二进制全是1，与hash进行与运算时，相当于对capacity取模，能够充分散列，减少冲突。

## 反射机制

1. 简单介绍一下java的反射机制？反射在哪些地方有应用场景？
  Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的以及动态调用对象的方法的功能称为Java的反射机制。三种获得class的方式：类名.class；Class.forName；实例.getClass；forName使用classLoader，并且初始化。
应用场景：几乎框架离不开反射，如Struts2，struts.xml配置了请求与具体action的一个方法的映射，view层的请求被拦截后，根据配置文件动态创建action实例。（动态代理）
动态类加载，比如我项目里对智能合约的支持使用了反射。

2. 反射创建类实例的两种方式是什么？
- class对象.newInstance()方法；
- 获取类的构造函数对象，getConstructor，使用构造函数对象的newInstance方法；

## 泛型

1. 在jdk1.5中，引入了泛型，泛型的存在是用来解决什么问题？
  泛型主要针对向下转型是带来的安全隐患，转换类型与对象类型不匹配，classcastexception，泛型能在编译期间进行安全检查，减少了强转的操作。泛型的本质是为了参数化类型。

2. 泛型擦除在哪个阶段？为什么要擦除？
  编译阶段，正确检验泛型结果后，将泛型相关信息擦除，在对象进入和离开方法的边界处添加类型检查和类型转换的方法。擦除的原因是为了与1.5之前的代码兼容

3. 既然object是根类 为啥还要泛型？
  1.5之前没有泛型，往集合中插入的都是object类型，需要程序员自己强制类型转换，安全隐患。有了泛型之后，在编译器就能进行类型检查，并且自动生成强转代码，减少出错率。

## 多线程

1. Java线程池四种介绍一下？实现？参数？
   newCachedThreadPool：可缓存线程池，线程创建多了，可灵活回收空闲线程
   newFixedThreadPool：控制线程最大并发数，超出的线程会在队列中等待
   newScheduledThreadPool：支持定时及周期性任务
   newSingleThreadExecutor：唯一的工作线程执行任务，保证任务的按照指定的顺序执行。
   尽量不适用Executors创建线程池，默认无界的队列，可能会导致OOM。使用ThreadPoolExecutor创建线程池有7个参数：
   - corePoolSize，线程池长期维持的线程数，当提交一个任务到线程池时，线程池会创建一个现成来执行任务，即使有其他空闲的基本线程，等到需要执行的任务数大于线程池基本大小时，就不再创建；
   - maximumPoolSize，最大线程数，如果任务队列满了，并且已创建的线程数小于最大线程数，则线程池再创建新的线程执行任务，对无界的任务队列这个参数无效；
   - keepAliveTime、unit，超过corePoolSize的线程的空闲时常，超过这个时间，线程会被回收；
   - workQueue，任务的队列，用于保存等待执行的任务的阻塞队列，有以下几种选择：
     - ArrayBlockingQueue，基于数组结构的有界阻塞队列，FIFO；
     - LinkedBlockingQueue，基于链表的阻塞队列，FIFO，newFixedThreadPool使用该队列；
     - SynchronousQueue，无容量，对队列的插入操作必须等另一个线程消费（读取/移除）操作，否则插入一直处于阻塞状态，newCachedThreadPool使用这个队列；
   - threadFactory，新线程的产生方式
   - handler，拒绝策略，超过队列大小，如何处理新来的任务；

2. 有哪些拒绝策略？
  任务之所以被拒绝添加到线程池，线程池异常关闭，或者任务数量超过线程池的最大限制。
  - DiscardPolicy，直接丢弃被拒绝的任务；
  - DiscardOldestPolicy，丢弃最旧的任务；
  - AbortPolicy，被拒绝时，抛出RejectExecutionException异常；
  - CallerRunsPolicy，不进入线程池，任务由调用者线程去执行；

3. ThreadPoolExecutor内部的工作原理是怎样的？
总结起来就是：当poolSize小于corePoolSize时，创建新的线程执行任务；当poolSize大于corePoolSize，等待队列未满时，则任务进入等待队列；当poolSize大于corePoolSize并且小于maximumPoolSize，且等待队列已满，则创建新的线程处理任务；当poolSize大于maximumPoolSize，且等待队列已满，则根据拒绝策略来处理该任务；另外，每个线程执行完任务后不会立即退出，而是去检查等待队列里是否还有线程任务，如果在keepAliveTime里等不到，线程被回收。

4. volatile关键字的作用？
  JVM中最轻量级的同步机制，具有可见性和有序性两种特性。对一个volatile变量的读总是能看到任意线程对这个变量最新的写入，线程在工作内存中修改该变量后立即更新到主存上。lock前缀指令生成一个内存屏障，保证重排序的代码不会越过内存屏障

5. 线程加锁有哪些方式？synchronized和lock的区别？
  加锁方式有synchronized，各种Lock。synchronized是Java关键字，在jvm层面上，通过对象头来实现加锁释放锁。Java中每一个对象都可以作为锁，synchronized修饰普通成员方法，锁是当前对象实例；synchronized修饰静态方法，锁是当前类的class对象；同步方法块，锁是synchronized括号里的对象。当一个线程访问同步代码块时，首先需要获得锁，退出或者抛出异常时必须要释放锁。同步代码块底层使用monitorenter和moniterexit指令实现。
  锁有读写锁，可重入锁等，是一个类。
  区别：synchronized无法判断锁状态，lock可以用trylock判断能不能获得锁；线程等待lock释放的过程可以用interrupt中断等待，而synchronizd只能等待锁释放。synchronized以发生异常的时候会自动释放锁，而lock不行。lock可以实现绑定多个条件，和公平锁，synchronized锁是非公平锁。

6. 介绍Java内存模型
  ![JMM](/images/20190829/jmm.png){:	.align-center}
  Java内存模型屏蔽系统和硬件的差异，主要划分为主内存和工作内存两种，分别对应物理内存，寄存器和高速缓存。每条线程拥有各自的工作内存，工作内存中的变量是主内存中的一份拷贝。

7. Java如何实现线程？
  - 继承Thread类创建线程
  - 实现Runnable接口
  - 实现Callable接口
  - 使用线程池，ThreadPoolExecutor

8. Java内存模型如何保证一致性？
  提供三个关键字由程序员控制，分别是：
  final：构造函数退出时，final域的值是被保证对其他线程可见的；
  volatile：可见性，屏蔽指令重排序；
  synchronized：原子性，可见性，屏蔽指令重排序；
  
9. volatile修饰的long或double类型能保证原子性。
对于64位的long和double，如果没有volatile修饰，一次写入会被分为两次分别针对高32位低32位的写操作，可能会导致高低32位被不同线程修改的问题。如果用volatile修饰，读写都是原子的。对64位的引用地址的读写也都是原子的。

10. CountDownLatch 和 CyclicBarrier 有什么区别？

### ThreadLocal

1. ThreadLocal到底有什么用？
  每个线程会在堆山开辟一块工作内存，而ThreadLocal就是线程工作内存中的一小块内存，用于存储线程独享的数据。相当于数据存储在线程本地，读写效率高，不必在主内存和工作内存之间来回复制。
  
2. ThreadLocalMap的key为什么要设置为弱引用？
  线程可以存储多个ThreadLocal变量，假设线程维护很多ThreadLocal，并且entry的key与threadlocal对象之间是强引用关系，当threadlocal的引用（栈中）置为null后，threadlocal并不会被GC回收，可能会造成堆上的OOM。而弱引用的作用就是关联的对象在下一次GC时会被回收，当ThreadLocal变量的强引用释放后，Entry的弱引用的key在下一次GC就会被回收。

![threadlocalmap](/images/20190829/threadlocalmap.jpg)

3. ThreadLocal中的内存泄漏问题。
key回收后，value并没有被回收，而这块内存永远不会被访问到，所以存在内存泄漏。如何避免这个问题？在调用ThreadLocalMap的get和set方法时，把key为null的的entry的value设置为null，或者调用remove。但是当不再去调用set get remove时，将导致内存泄漏。
另一种情况就是ThreadLocal变量为static时，线程结束时，static变量不一定会被回收，更容易导致内存泄漏。

4. 多线程内存泄漏问题？
如3，value的生命周期最长可以跟线程一致，线程不被回收，就存在内存泄漏的问题。当使用线程池时，线程结束后，放回线程池中不被销毁，线程一直不被使用或者不再调用get set方法，就会内存泄漏。


5. ThreadLocal使用场景？
  最常见的场景使用ThreadLocal解决数据库连接或者session管理等。

### CAS详解

1. CAS是什么？
CAS（compareAndSwap）是一条CPU并发原语，与悲观锁相对，是一种乐观的并发放是，可以原子地完成以下操作：判断内存某个地址的值是否为预期值，如果是则更新该值。

2. Java实现CAS，其底层原理是什么？
CAS依赖于CPU提供的cas指定，如Intel的cmpxchg指令，由于Java无法直接访问底层系统，其依赖于Unsafe提供的native方法实现。以`AtomicInteger`为例，cas一定与volatile变量搭配使用，保证每次拿到主内存中的最新值。unsafe根据value在内存中的偏移地址获取数据，不断循环直至CAS成功。
```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
}
```

3. CAS存在什么缺陷或者问题？如何解决？
  - 长时间自旋开销大，CAS一直失败；
  - 只能保证对一个共享变量的原子操作，多个共享变量的操作智能使用加锁来保证；JDK1.5之后，可以使用AtomicReference类保证引用对象之间的原子性，把多个变量放在一个对象中进行CAS操作；
  - 存在ABA问题，即一个线程在获取当前变量时，另外线程两次修改改值，先变为B在变为A，第一个线程仍旧认为CAS成功，实际有线程在改变它。解决方案是JDK1.5之后提供一个AtomicStampReference，数据加上一个版本号；
  
# JVM

## 内存

1. 什么情况下会发生内存溢出？
  程序计数器是唯一一个在Java虚拟机规范中没有规定任何OOM情况的区域。
- a. 动态可扩展的虚拟机栈，在无法申请到足够的内存是会OOM；
- b. 与虚拟机栈一样，本地方法栈同样会OOM；
- c. 如果堆中没有足够内存完成实例分配，且堆无法扩展时会OOM；
- d. 产生大量的类，方法区保存class的相关信息，大量的类会使得方法区OOM；
- e. 运行时常量池OOM，过多的常量字符串造成OOM；

2. JVM的内存区域？

  ![JMM](/images/20190829/jmm.jpeg){:.align-center}

线程共享的有方法区和堆，线程私有的为虚拟机栈，PC程序计数器和本地方法栈。
虚拟机栈：描述Java方法执行的内存模型，每个方法被执行的时候都会创建一个栈帧，包括局部变量表，操作栈，动态链接和方法出口。栈帧的大小在编译时就确定。
所有对象实例以及数组在堆上分配内存。
方法区存储类信息（类名，访问修饰符，字段描述，方法描述），常量，静态变量。JDK7以后，运行时常量池移到堆上。

3. JVM内存为什么要分为新生代，老年代，持久代。新生代为什么要分eden区与survivor区？
  分代的垃圾回收策略是基于这样一个事实：不同对象的生命周期是不一样的。不同生命周期的对象可以采取不同的收集方式，以提高回收效率。如果不进行分代，每次垃圾回收都对整个堆进行回收，效率很低。有些对象例如session对象，socket连接生命周期比较长；临时变量，生命周期短。
  如果没有survivor区，一次minor GC过后，存活对象晋升到老年代，老年代很快填满，触发full gc。survivor存在的意义就是控制进入老年代的对象，进而减少full gc的发生。新生代对象一般朝生夕死，每次回收都会发现大量死去的对象，采用效率较高的复制算法，复制存活的对象到另一块内存，由于存活对象一般很少，所以分为eden区和survivor区，且默认比例为8:1。当eden区剩余内存不足以分配对象，进行一次minor GC，将存活对象复制到一块survivor上。使用两块survivor可避免内存碎片化，第一次minor gc后，eden区存活对象进入一块survivor区，再一次minor gc后，Eden和s0的存活对象进入另一块survivor，占用连续空间，避免碎片。

4. JVM中一次完整的GC流程是怎样的，对象如何晋升到老年代，说说你知道的几种主要的JVM参数。
  对象优先在eden区进行分配，大于pretenuredthreshold的对象直接进入老年代。如果eden区没有足够内存进行分配，则进行一次minor gc，将eden区和一块survivor区的存活对象转移到另一块survivor区。如果此时发现survivor区没有足够内存容纳所有的存活对象，则通过分配担保机制将无法容纳的对象转移到老年代。对象在survivor区中每熬过一次minor gc年龄就增长一岁，默认超过15岁（maxtenuringthreshold）的对象直接转移到老年代。还有一种情况，survivor空间中相同年龄的所有对象大小总和大于survivor空间的一半，年龄>=该年龄的对象直接进入老年代。如果老年代没有足够空间容纳转移过来的对象，则进行一次full gc。ful gc对整个堆进行一次gc。

  - -Xms -Xmx：最小堆内存，最大堆内存
  - SurvivorRatio：Eden与survivor比例
  - maxtenuringthreshold：转移到老年代的年龄阈值
  - pretenuredthreshold：大于该阈值的对象直接进入老年代
  - useadaptivesizepolicy：parallel scavenge自适应调整新生代大小，eden survivor比例等细节参数。
  - cmsinitiatingoccupancyfraction：触发CMS收集器的比例

5. 你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms和G1，包括原理，流程，优缺点。

   |        收集器        |              简单描述              |     优点      |    缺点    |
   | :---------------: | :----------------------------: | :---------: | :------: |
   |      Serial       |   新生代收集器，停止所有工作线程，复制算法单线程收集    |    简单高效     | 需要停止工作线程 |
   |      ParNew       |          Serial的多线程版本          |  多线程环境下高效   |  停止工作线程  |
   | Parallel Scavenge | 新生代收集器，专注于吞吐量，适合后台计算不需要太多交互的任务 | 自适应开关动态调整参数 |  停顿时间变长  |
   |    Serial Old     | 老年代收集器，单线程，标记整理，作为CMS失败时的后备预案  |             |          |
   |   Parallel Old    |      老年代收集器，与Scavenge搭配使用      |     吞吐高     |          |
CMS是一种以获取最短回收停顿时间为目标的收集器，其原理是耗时最长的并发标记与并发清除步骤是与工作线程并发执行的，所以能缩短停顿时间。初始标记非常简单，仅仅标记以下GC Roots能直接关联到的对象，并发标记就是GR Roots追踪的过程，而重新标记是修正并发标记过程中因用户程序继续运行而导致标记产生变动的那部分对象，最后并发清除标记的对象即可。其优点是并发收集并且停顿时间短。缺点如下：对CPU资源敏感，占据一部分资源导致用户程序变慢，CPU数量不足的情况下影响较大；无法处理浮动垃圾，由于并发清除过程中用户程序还在运行，产生新的垃圾，如果CMS预留的内存无法满足用户程序需要，则产生一次Concurrent Mode Failure，启用Serial Old备案，重新进行老年代收集，这样停顿时间就很长。参数：CMSInitiatingOccupancyFraction；最后一个缺点，基于标记清除的算法会产生内存碎片，参数UseCMSCompactAtFullCollection。
Garbage-First，从JDK9开始G1为默认的垃圾收集器。G1将堆划分为规整的region，保留了新生代与老年代的分代策略，但是他们不一定要物理上连续，即新生代是一堆region的集合。并行与并发，分代收集，空间整合，可预测的停顿时间。根据允许的收集时间优先回收价值最大的region。

6. 当出现了内存溢出，你怎么排错？
  heapdumponoutofmemory，对dump出来的堆转储快照进行分析，内存泄漏还是内存溢出

7. JMM Java内存模型了解吗？比如重排序，内存屏障，happen-before，主内存，工作内存等。
  TODO

8. 怎么打印线程栈信息？用过哪些命令查看jvm的状态、堆栈信息？
  jps获得进程号；
  top pid获得本进程中的所有线程
  jstack命令查看当前java进程的堆栈状态
  
9. 哪些对象可以作为GC Root？
  - 虚拟机栈（栈帧中的局部变量表）中引用的对象；
  - 方法区中类静态属性引用的对象；
  - 方法区中常量引用的对象；
  - 本地方法栈中native方法引用的对象；

## 类加载

1. 简单说说你了解的类加载器，可以打破双亲委派么，怎么打破。
  类加载器就是根据给定的全限定名将class文件加载到内存，并对数据进行校验，准备，解析和初始化，最终转为一个class对象。有如下几类加载器：
- 启动类加载器，负责将JAVA_HOME\lib下或者bootclasspath指定路径下的类加载到虚拟机中；
- 扩展类加载器，负责将\lib\ext下或者由java.ext.dirs系统变量指定路径下的类加载到虚拟机中；
- 应用程序类加载器，负责加载用户类路径classpath下指定的类，一般情况为程序默认的类加载器；
- 自定义类加载器
双亲委派模型：如果一个类加载器收到一个加载请求，它首先不会去尝试加载这个类，而是把这个请求委派给父类加载器完成。只有当父加载器在自己的搜索范围内找不到指定类时，子加载器调用findClass尝试自己去加载。
打破双亲委派模式的一种场景是，基础类需要回调用户代码的时候，引入一个线程上下文类加载器，父加载器可以委托子加载器去完成类加载请求。
或者重写loadClass和findClass方法，因为默认的loadClass是双亲委派的实现。

2. 动态加载类的框架了解哪些？
  Struts、Spring等

3. 动态代理一般有哪几种实现方式？动态代理的应用场景有哪些？
  Proxy+InvocationHandler实现，一个拦截器。接口代理
  cglib，子类代理
  场景：调用和扩展被代理类的方法，比如加日志，struts2中action的调用，spring的AOP。

4. java类加载机制？如何实现自定义类加载器？findClass与loadClass的区别？
  类加载机制如1。实现自定义加载器，在不到破双亲委派机制前提下，重写findClass方法即可，若打破，两个都需要重写。findClass是当前加载器在搜索范围内查找类的方法，默认的loadClass实现双亲委派，在loadClass方法中，若父加载器无法找到类，则调用自身加载器的findClass方法。
  
5. 类加载的几个步骤？
  - 加载：把类的class文件加载到内存，产生Class对象。
  - 连接：验证字节码，为静态变量分配内存设置默认初始值（准备），将符号引用替换为直接引用（解析）。
  - 初始化：执行clinit方法，父类先执行，子类后执行，该方法为编译器自动收集类中所有类变量的赋值和静态语句块。

## 编译优化

1. Java程序是如何执行的？
  - 先把Java代码编译成字节码（class文件），即通过词法分析，语法分析，语义分析最后使用字节码生成器生成最终的class文件；
  - JVM使用类加载器将字节码文件加载进内存，包括一些列的字节码检查，连接，初始化等操作；
  - 执行的时候，解释器将字节码翻译成字节码进行执行。但是不是所有的代码都通过解释器解释执行，HotSpot虚拟机提供JIT，将热点代码翻译成机器码，是一种动态编译执行的方式；

2. 介绍两种热点判定方式。
  - 基于采样的热点探测：虚拟机周期性地检查各个线程的虚拟机栈顶，若某个方法经常出现在栈顶，那这个方法就是热点方法。优点是简单高效，缺点是不精确，容易收到线程阻塞或外界因素干扰；
  - 基于计数器的热点探测：为每个方法（代码块）建立计数器，统计执行次数，达到阈值的方法被认为是热点代码；分为两种计数器，方法调用计数器，顾名思义针对方法调用的，回边计数器针对循环体代码块的。两种方法的均以整个方法对编译对象，不同的是，由于循环体的编译发生在执行过程中给，即方法栈帧还在栈上，称为栈上替换OSR。

3. 有哪些编译优化技术？

