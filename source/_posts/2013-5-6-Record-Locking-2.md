layout: post
title: Record Locking 2
date: 2013-5-6
categories: Linux
---

之前分析过record locking/file locking读锁和写锁的概念和Linux API。
当进程都按加锁的机制访问文件时，记录锁能保证其正确访问。
但是如果对于同一文件，有的进程加锁访问，有的进程不加锁访问，这种情况操作系统怎样处理？
因此今天再分析两种记录锁的策略：

- 协同锁 Advisory
- 强制锁 Mandatory

### 协同锁 Advisory

协同锁降低了访问共享文件的限制，使得没有加锁的进程仍然能够访问共享文件。
也就是说操作系统不对无锁的进程进行限制。
因此想要正确处理共享文件的访问，需要用户主动提供协同加锁访问的进程。

大多数UNIX和UNIX-like的操作系统默认提供Advisory类型的锁策略。
也可以通过修改mount参数提供Mandatory类型的锁策略。

Java提供了Advisory类型的锁策略。

通过查看`/proc/locks`文件可以看到当前操作系统中存在的锁：

```sh
>cat /proc/locks
1: FLOCK  ADVISORY  WRITE 157 00:0e:7976 0 EOF
```

第二列说明了当前锁的策略，第三列说明锁的类型。

### 强制锁 Mandatory

强制锁利用操作系统对共享文件的访问做限制，使得无锁进程不能访问加锁的共享文件（阻塞）。
操作系统在此做强制性限制。

Microsoft操作系统提供Mandatory类型的锁策略。


### 单实例程序

文件锁可以用来实现单实例程序，如单实例的daemon。
这通常放在`/var/run`目录下，以`.pid`结尾。

### Reference

1. www.thegeekstuff.com/2012/04/linux-file-locking-types/

2. Java NIO Book
