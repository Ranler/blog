layout: post
title: Record Locking
date: 2013-4-29
categories: Linux
---

**记录锁(Record Locking)/文件锁(File Locking)**的功能是：当一个进程正在读或修改文件的某个部份时，
它可以阻止其它**进程**修改同一文件区。
记录锁是面向进程的，因此不能用于线程间。
记录锁是一种字节范围锁(byte-range locking)，它锁定的只是文件中的一个区域（也可以是整个文件）。

记录锁可用于实现进程间上锁，尤其是无亲缘关系的进程间上锁，而不是用于同一进程下不同线程间的上锁。

System Call:

- `flock()`, which places lock on entire files，创建文件锁
- `fcntl()`, which places lock on regions of file，创建记录锁

一个值得注意的是：**父进程设置的记录锁不会被子进程继承**。


### `fcntl()`

记录锁主要使用的OS API是`fcntl`。
`fcntl`的接口是：

```c
#include <fcntl.h>
int fcntl(int filedes, int cmd, ...);
```

`fcntl`函数可以改变已打开的文件性质。
第一个参数是目标文件的文件描述符。
第二个参数是操作标记，可以有以下5种功能：

1. 复制一个现有的描述符(cmd=F_UDPFD)
2. 获得/设置文件描述符标记(cmd=F_GETFD或F_SETFD)
3. 获得/设置文件状态标志(cmd=F_GETFL或F_SETFL)
4. 获得/设置异步I/O所有权(cmd=F_GETOWN或F_SETOWN)
5. 获得/设置记录锁(cmd=F_GETLK, F_SETLK或F_SETLKW)

前4种功能主要是文件操作，第三个参数总是一个整数。
第5种功能是本文关注的记录锁功能，第三个参数是一个指向flcok结构的指针。

```c
struct flock {
	short	l_type;
	short	l_whence;
	__kernel_off_t	l_start;
	__kernel_off_t	l_len;
	__kernel_pid_t	l_pid;
	__ARCH_FLOCK_PAD
};
```

flock结构各成员说明如下：

- l_type：锁的类型：
  - F_RELCK 共享读锁/shared lock
  - F_WRLCK 独占写锁/exclusive lock
  - F_UNLCK 解锁
- l_whence：加锁/解锁区域偏移起点：
  - SEEK_SET：文件开始处
  - SEEK_CUR：文件当前偏移处
  - SEEK_END：文件当前长度处，即文件末尾
- l_start：加锁/解锁区域相对l_whence起始字节偏移量
- l_len：加锁/解锁区域长度

l_whence和l_start的组合与其在`fseek`函数中很像。
如果要锁整个文件，那么`l_whence=SEEK_SET,l_start=0,l_len=0`即可。

### 锁的类型

和一般锁的定义一样，读锁可以共享，写锁必需独占。
一个文件区域某一时刻只能有一把锁。
如果一个进程对一个文件区域已经有一把锁，后来该进程又企图在同一文件区域再加一把锁，
那么新锁将替换老锁。

通过`fcntl()`获得的lock时是inode和process关联的。
当关闭一个文件描述符时，那么其关联的inode上该进程设置的锁将被释放。
因此通过不同`open()`打开的文件描述符，如果通过其中一个加锁，然后`close()`另外一个文件描述符，那么锁仍将会被释放。这个和`flock()`不同。
`fcntl()`获得的锁不能被继承。

加读锁时，文件描述符必需是读打开。
加写锁时，文件描述符必需是写打开。



### 锁的竞争方式

上面说了`fcntl`的`cmd`参数可以设置三种参数，它们的作用分别是：

##### F_GETLK

测试flock指定的锁能否获取

- 如果能，把l_type设为F_UNLCK。
- 如果不能，把正在占用文件区域的锁的信息写入flock。

##### F_SETLK

非阻塞竞争或释放flock指定的锁，

- 如果成功，fcntl返回0
- 如果失败，fcntl立即返回-1并设置errno为EAGAIN

对于想建立非阻塞锁的程序，应该使用F_SETLK，并对返回结果测试。

##### F_SETLKW

阻塞竞争flock指定的锁，直到竞争成功返回。

关于记录锁的三条规则：

1. 关闭文件描述符时，锁都被释放
2. fork产生的子进程不继承父进程的锁
3. exec后，新程序可以继承原进程的锁，除了文件描述符设置了close-on-exit。

对于待处理的上锁请求是怎样处理的（是FIFO还是有优先考虑读锁），Poxis标准并未说明。

### `flock()`

`flock()`来自于BSD。它的作用是锁住整个文件。

```c
#include <sys/file.h>
int flock(int fd, int operation);
```

A file lock obtained via `flock()` is associated with the **open file description(system-wide)**, rather than the file descripor(process-wide) or the file(i-node)(system-wide) itself. This is different with record locks obtained by `fcntl()`。

而通过`dup()`,`dup2()`和`fcntl()`拷贝的file descriptor是共享open file description。
因此这些拷贝的file descriptor只需其中之一释放锁即可。如果都不显式释放，那么当所有拷贝的file descriptor关闭后，open file description上的锁就会自动释放。
同样，`fork()`也是通过`dup()`复制文件描述符，也就是说其可以被其它进程继承。

而通过`open()`打开同一文件时，每次打开都会创建新的open file description。
每个open file description都可绑定独立的锁。


### Mix `stdio` and locking

Because of the user-space buffering performed by the `stdio` library,
we should be cautious when using `stdio` functions with the file lock.
The problem is that an input buffer might be filled before a lock is placed,
or an output buffer might be flushed afther a lock is removed.
There are some ways to avoid these problems:

- Using system call `read()` and `write()` install of the `stdio` library.
- Flush the stdio stream immediately after placing a lock on the file, and flush it once more immediately before releasing the lock.
- Perhaps at the cost of some efficiency, disable stdio buffering altogether using `setbuff()`(or similar).

### Reference

- APUE
- The Linux Programming Interface

