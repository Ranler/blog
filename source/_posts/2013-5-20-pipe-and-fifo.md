layout: post
title: PIPE and FIFO
date: 2013-5-20
categories: Linux
---

- PIPE: (无名)管道，通常用于亲缘关系的进程间
- FIFO: 命名管道，可用于无亲缘关系的进程间

PIPE和FIFO都通过`read()`和`write()`函数访问。

### PIPE

```
#include <unistd.h>
int pipe(int fd[2]);
```

原始的PIPE是半双共工。
PIPE常用于父子(fork)进程间数据的**单向**发送，即半双工。
当需要一个全双工的数据流时，必需创建两个管道，每个方向一个。

现在很多UNIX都提供全双工的PIPE。
因此只需创建一个管道即可实现全双工的数据流。

### FIFO

```
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char* pathname, mode_t mode);
```

FIFO用于实现命名的半双工数据流。
用户程序可以通过`open()`打开指定路径的FIFO文件进行读或者写。
`read()`总是读取数据的开头，`write()`总是写入数据的末尾。
当没有任何进程打开FIFO写时，那么打开该FIFO来读的进程将阻塞。

注意：如果PIPE写入的字节数小于PIPE_BUF，那么write会保证是**原子**的。
是否block对此也无影响。
但是如果write写入的字节数大于PIPE_BUF，那么write不保证是原子的。



