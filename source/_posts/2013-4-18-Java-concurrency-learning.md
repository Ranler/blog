layout: post
title: Java Concurrency 学习
date: 2013-4-18
categories: JVM
---

并发机制：

- 内存共享: Java, C#, Pthread, win32
- 消息传递：JVM:Scale, Google:Go, Ericsson:Erlang

特点：

- 内存共享：显示同步，隐式通信
- 消息传递：显示通信，隐式同步

Java并发＝显示同步＋隐式通信

Java并发处理对象（Shared+Mutable对象）：

- 共享变量（需要处理）：实例域，静态域，数组元素
- 局部变量（不需处理）：局部变量，方法定义参数，异常处理参数

还有一个危险来源于64位类型long和double的非原子操作。

并发编程面对的问题：

- visibility
- ordering
- atomicity

### 指令重排序 Reordering

为了提高指令处理效率，存在以下三个级别的指令重排序：

- 编译器级(软): just-in-time compiler and bytecode compiler
- 指令级并行级(硬): Processer
- 内存系统级(硬): memory hierarchy

重排序的首要条件：保证数据依赖关系。

硬件(CPU)重排序类型，第一种所有平台都支持，后三种选择性支持：

- 写读(都支持)
- 写写
- 读写
- 读读

为了在硬件重排序的情况下也能保证内存可见性，需要在两个指令中间插入内存屏障指令。
内存屏障指令会保证第一个操作的对象写到主存，并把其它线程中引用该对象的内存设为失效。

### Java简单内存模型

Java Memory Model(JMM)


### synchronized 内置锁/监视器锁

- 锁保护的代码块
- 锁的对象：方法所在的this对象（一般方法）或Class对象（静态方法）

对象内置锁称为Monitor，每个对象一个。

Java获取锁的操作的粒度是线程，而不是调用。
就是说同时只能有一个线程进入同一个锁，其它想获得此锁的线程会阻塞。
并且这个线程对于此锁是可重入的，便于支持锁方法的继承。
每个锁记录了获取次数和一个所有者线程。

之所以每个对象都有一个内置锁，只是为了免去显示创建锁对象。
常见的加锁策略是把所有可变状态封装到对象内部，利用内置锁对访问这些可变状态的线程进行同步。

通过显示地加synchronized锁，JMM会隐式通过通信机制保证：

- 释放锁时，临界区的共享变量从本底内存同步到主内存
- 获取锁时，临接区的共享变量被置为无效，即再从主内存同步到本底内存

synchronized会引起上下文线程切换(?)。

key ideas about synchronized:

- **Conflicting Access**: at least one of accesses is a write
- **Happen-Before Relationship**: If one action happen before another, then the first is visible to and ordered before the second.Including:
  - Each action in a thread happens before every subsequent action in that thread.
  - An unlock on a monitor happens before every subsequent lock on that monitor.
  - A write to a volatile field happens before every subsequent read of that volatile.
  - A call to `start()` on a thread happens before any actions in the started thread.
  - All actions in a thread happen before any other thread successfully returns from a `join()` on that thread.
  - If an action a happens before an action b, and b happens before an action c, then a happens before c.

### volatile

volatile是轻量级同步机制，也是由JMM隐式通信机制保证。
访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞。

- volatile变量写时：从本地内存同步到主内存，标记其它线程变量失效
- volatile变量读时：如果失效，从主内存同步到本地内存

volatile防止了指令重排序。

volatile常用于某个操作完成，发生中断或状态的标志。
但是volatile变量只能保证可见性，不能保证原子性。
如volatile的count++的原子性就不能保证。

### final

除了final限制了成员的不变性之外，
final成员和普通成员的区别是：final成员必需在新对象构造函数中赋值完成。

需要这点保证的直接原因是新对象的引用和初使化（构造函数）指令可能会重排序。
也就是说可能新对象的引用先被赋于某个变量，然后再使用构造函数初使化新对象成员，
直到第一次使用新对象成员之前完成即可。
这种情况在Java和CPP中都存在。
这样如果令一个线程在构造函数完成之前访问新对象的成员，就会是一个无效值。

因此，final成员在初次获得对象的引用之后（也就是构造函数完成之后），就保证了能在令一个线程正确读取到final值。这两个顺序是有保证的。

### Reference

- JLS Chapter 17
- JSR 133
