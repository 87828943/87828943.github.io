---
title: ConcurrentHashMap原理
date: 2018-07-05 10:47:19
tags: [高并发,ConcurrentHashMap,HashMap]
---

由于 `HashMap` 是一个线程不安全的容器，主要体现并发情况下在size大于`容量*负载因子`发生扩容时，取一个不存在的 `key` 时，计算出的 `index` 正好是环形链表的下标会出现环形链表从而导致死循环。
因此需要支持线程安全的并发容器 `ConcurrentHashMap` 。

> 所以 `HashMap` 只能在单线程中使用，并且尽量的预设容量，尽可能的减少扩容。
> 在 `JDK1.8` 中对 `HashMap` 进行了优化： 当 `hash` 碰撞之后写入链表的长度超过了阈值(默认为8)，链表将会转换为红黑树。
> 假设 `hash` 冲突非常严重，一个数组后面接了很长的链表，此时重新的时间复杂度就是 `O(n)` 。
> 如果是红黑树，时间复杂度就是 `O(logn)` 。
> 大大提高了查询效率。

## 数据结构

![](/images/2018-07-05/concurrentHashMap.jpg)

<!-- more -->

如图所示，是由 `Segment` 数组、`HashEntry` 数组组成，和 `HashMap` 一样，仍然是数组加链表组成。

`ConcurrentHashMap` 采用了分段锁技术，其中 `Segment` 继承于 `ReentrantLock`。不会像 `HashTable` 那样不管是 `put` 还是 `get` 操作都需要做同步处理，理论上 `ConcurrentHashMap` 支持 `CurrencyLevel (Segment 数组数量)`的线程并发。每当一个线程占用锁访问一个 `Segment` 时，不会影响到其他的 `Segment`。

## GET方法

`ConcurrentHashMap` 的 `get` 方法是非常高效的，因为整个过程都不需要加锁。

只需要将 `Key` 通过 `Hash` 之后定位到具体的 `Segment` ，再通过一次 `Hash` 定位到具体的元素上。由于 `HashEntry` 中的 `value` 属性是用 `volatile` 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。

## PUT方法

内部 `HashEntry` 类 ：
```java
	static final class HashEntry<K,V> {
	    final int hash;
	    final K key;
	    volatile V value;
	    volatile HashEntry<K,V> next;

	    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
	        this.hash = hash;
	        this.key = key;
	        this.value = value;
	        this.next = next;
	    }
	}
```
虽然 `HashEntry` 中的 `value` 是用 `volatile` 关键词修饰的，但是并不能保证并发的原子性，所以 put 操作时仍然需要加锁处理。

首先也是通过 Key 的 Hash 定位到具体的 Segment，在 put 之前会进行一次扩容校验。这里比 HashMap 要好的一点是：HashMap 是插入元素之后再看是否需要扩容，有可能扩容之后后续就没有插入就浪费了本次扩容(扩容非常消耗性能)。

而 ConcurrentHashMap 不一样，它是先将数据插入之后再检查是否需要扩容，之后再做插入。

## SIZE方法

每个 `Segment` 都有一个 `volatile` 修饰的全局变量 `count` ,求整个 `ConcurrentHashMap` 的 `size` 时很明显就是将所有的 `count` 累加即可。但是 `volatile`修饰的变量却不能保证多线程的原子性，所有直接累加很容易出现并发问题。

但如果每次调用 size 方法将其余的修改操作加锁效率也很低。所以做法是先尝试两次将 `count` 累加，如果容器的 `count` 发生了变化再加锁来统计 `size`。

至于 `ConcurrentHashMap` 是如何知道在统计时大小发生了变化呢，每个 `Segment` 都有一个 `modCount` 变量，每当进行一次 `put remove` 等操作，`modCount` 将会 +1。只要 `modCount`发生了变化就认为容器的大小也在发生变化。