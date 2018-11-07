---
title: volatile 关键字
date: 2018-07-05 11:11:39
tags: [高并发,volatile]
---

不管是在面试还是实际开发中 volatile 都是一个应该掌握的技能。

> 内存可见性&指令重排

## 内存可见性

由于 `Java` 内存模型(`JMM`)规定，所有的变量都存放在主内存中，而每个线程都有着自己的工作内存(高速缓存)。

线程在工作时，需要将主内存中的数据拷贝到工作内存中。这样对数据的任何操作都是基于工作内存(效率提高)，并且不能直接操作主内存以及其他线程工作内存中的数据，之后再将更新之后的数据刷新到主内存中。

> 这里所提到的主内存可以简单认为是**堆内存**，而工作内存则可以认为是**栈内存**。

<!-- more -->

如下图所示：

![](/images/2018-07-05/volatile.jpg)

所以在并发运行时可能会出现线程 B 所读取到的数据是线程 A 更新之前的数据。

显然这肯定是会出问题的，因此 `volatile` 的作用出现了：

> 当一个变量被 volatile 修饰时，任何线程对它的写操作都会立即刷新到主内存中，并且会强制让缓存了该变量的线程中的数据清空，必须从主内存重新读取最新数据。

`volatile` 修饰之后并不是让线程直接从主内存中获取数据，依然需要将变量拷贝到工作内存中。

**`volatile` 保证了内存可见性，每个线程拿到的值都是最新值，并不能保证线程安全性！**

* 所以想到达到线程安全可以使这三个线程串行执行(其实就是单线程，没有发挥多线程的优势)。

* 也可以使用 synchronize 或者是锁的方式来保证原子性。

* 还可以用 Atomic 包中 AtomicInteger 来替换 int，它利用了 CAS 算法来保证了原子性。

## 禁止重排序

内存可见性只是 `volatile` 的其中一个语义，它还可以防止 `JVM` 进行指令重排优化。

举一个伪代码:
```java
int a=10 ;//1
int b=20 ;//2
int c= a+b ;//3
```
一段特别简单的代码，理想情况下它的执行顺序是：`1>2>3`。但有可能经过 JVM 优化之后的执行顺序变为了 `2>1>3`。

可以发现不管 JVM 怎么优化，前提都是保证单线程中最终结果不变的情况下进行的。

可能这里还看不出有什么问题，那看下一段伪代码:
```java
private static Map<String,String> value ;
private static volatile boolean flag = fasle ;

//以下方法发生在线程 A 中 初始化 Map
public void initMap(){
	//耗时操作
	value = getMapValue() ;//1
	flag = true ;//2
}


//发生在线程 B中 等到 Map 初始化成功进行其他操作
public void doSomeThing(){
	while(!flag){
		sleep() ;
	}
	//dosomething
	doSomeThing(value);
}
```
这里就能看出问题了，当 `flag` 没有被 `volatile` 修饰时，`JVM` 对 1 和 2 进行重排，导致 `value` 都还没有被初始化就有可能被线程 B 使用了。

所以加上 `volatile` 之后可以防止这样的重排优化，保证业务的正确性。

## 总结

`volatile` 在 `Java` 并发中用的很多，比如像 `Atomic` 包中的 `value`、以及 `AbstractQueuedLongSynchronizer` 中的 `state` 都是被定义为 `volatile` 来用于保证内存可见性。