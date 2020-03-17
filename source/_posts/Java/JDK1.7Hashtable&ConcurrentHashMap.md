---
title: JDK1.7Hashtable&ConcurrentHashMap探究
date: 2020-03-08 
author: codezhw

summary: 这是一篇关于Java JDK1.7版本,介绍实现Map线程安全的文章，对Hashtable、ConcurrentHashMap进行探究，分析它们实现线程安全的原理，以及二者之间的比较。
categories: Java
tags: 
    - HashMap
    - 源码
---





# JDK1.7Hashtable&ConcurrentHashMap探究





在JDK1.7HashMap探究&手写HashMap这篇文章中，我们探究了HashMap的底层实现原理，分析了产生死锁的原因。

HashMap在我们日常生活中使用频繁，不免在多线程的环境下使用，因为HashMap是非同步的，那么就很有可能产生死锁的情况。

有什么办法能解决这样的问题呢？

我们来看HashMap文档的注释。

```visual basic
(The <tt>HashMap</tt>
* class is roughly equivalent to <tt>Hashtable</tt>, except that it is
* unsynchronized and permits nulls.) 
```

这段话的意思是：HashMap大致相当于Hashtable, 除了它是不安全的以及允许为空。

结论：

> **如果我们想在多线程环境下使用HashMap，我们可以使用Hashtable来代替它，Hashtable大致相当于HashMap，只是key和value不允许为空。**



## Hashtable与HashMap的比较



我们来看一下Hashtable的源码

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable
```

首先Hashtable继承自Dictionary类，实现了Map、Cloneable以及Serializable接口

Dictionary 类是一个抽象类，用来存储键/值对，作用和Map类相似。

但是现在以及过时。

参数方面基本和HashMap一致。

**构造函数方面：**

```java
public Hashtable() {
    this(11, 0.75f);
}

public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        initHashSeedAsNeeded(initialCapacity);
    }
```

容器的默认初始容量为11，负载因子为0.75。



**计算hash值的方法**

```java
private int hash(Object k) {
    // hashSeed will be zero if alternative hashing is disabled.
    return hashSeed ^ k.hashCode();
}
```

直接调用了object的hashCode()方法然后与hashSeed做一次异或运算，这样的操作效率是很低的。

**扩容**

```java
int newCapacity = (oldCapacity << 1) + 1;
```

扩容后的新的数组容量 = （原数组容量*2） +1



- **总结**

> * **Hashmap中的默认初始容量为16，Hashtable中为11，负载因子都是0.75**
>* **HashMap中允许存储null值，Hashtable中会抛出异常。**
> * **HashMap中计算hash值的方法比较复杂，进行了多次右移运算，再进行运算得到hash值，Hashtable中比较简单，直接用hashcode()方法得到进行运算。**
> * **HashMap的扩容为原数组的两倍，Hashtable是两倍+1**





## **Hashtable的并发核心思想**

```java
public synchronized V put(K key, V value) 
public synchronized V remove(Object key) 
public synchronized V get(Object key)
```

可以看到这些关键操作都加上了synchronized关键字，我们都知道synchronized是重量级锁，系统检查到锁是重量级锁之后，会把等待想要获得锁的线程进行**阻塞**，被阻塞的线程不会消耗cup。但是阻塞或者唤醒一个线程时，都需要操作系统来帮忙，这就需要从**用户态**转换到**内核态**，而转换状态是需要消耗很多时间的，有可能比用户执行代码的时间还要长。

而且可以发现一个问题：

假设我们在多线程环境下对一个hashtable进行操作，我们put()一个数据进入下标为5的位置，同时想要对下标为3的数据进行读取操作，这在Hashtable中不能实现的，因为这些方法都加上了synchronized关键字，在同一时刻只能有一个线程进行操作，其他线程处于等待阻塞状态。

**这样的做法实现了并发安全性，却大大的降低了我们的效率。**

## Collections.synchronizedMap





JDK在Collections对象中提供了大量可以实现集合并发安全的方法：

![](https://i.loli.net/2020/03/08/NrQ1kFUWuJBocjT.png)

在HashMap文档中我们得知利用Collections.synchronizedMap方法传入一个HashMap实例去得到一个并发安全的map。

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
    return new SynchronizedMap<>(m);
}

//SynchronizedMap是一个静态内部类
private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable{
    //省略
}
```

![synchronizedMap](https://i.loli.net/2020/03/08/piw8R2BhcAtxJCq.png)

**它里面的大多数操作也都是加入synchronized关键字，基本上和Hashtable的原理一样，所以它的效率也是很低的。**

## ConcurrentHashMap

**问题：**

> **HashTable容器在竞争激烈的并发环境下表现出效率低下的原因是所有访问HashTable的线程都必须竞争同一把锁，假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效提高并 发访问效率，这就是ConcurrentHashMap所使用的锁分段技术**



在 JDK1.5中 Java提供了 java.util.concurrent.ConcurrentHashMap 类，首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。



先暂时放在这里。



