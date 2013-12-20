---
layout: default
title: Kernel and User Process Communication
---

之前了解IPC很多，IPC用于用户进程之前交换数据。
而用户进程与内核的通信用IPC那些接口就不行了。
今天学习一下用户进程与内核的通信方法。

### 1. Kernel 启动参数

这种方法比较常用，比如在`grub`中修改向内核传递的参数。

用户可以使用内核提供的宏`__setup("para_name=", parse_func)`实现传递参数的解析，
并把传递进来的值转换成相应的内核变量的值，并设置那个内核变量。
然后把源码加入到内核源码中一同编译，就可以实现向内核传递信息的方法。

### 2. 模块参数和Sysfs

对于内核模块来说，声明为static的变量可以通过命令行来设置：

```
insmod ./module-ex.so para-ex=100
```

利用[Sysfs](http://en.wikipedia.org/wiki/Sysfs)也可以设置内核参数。
Sysfs是基于ramfs的文件系统(我的Archlinux已经挂载)，其对用户提供了一种按文件操作读取和写入内核参数的方法。
同样对于内核模块来说，通过`module_param`宏显式声明static变量，也可以在Sysfs中访问。

### 3. sysctl 命令

[Procfs](http://en.wikipedia.org/wiki/Sysfs)下也向用户提供了内核参数的文件表示。
`sysctl`命令可以修改`/proc/sys`下列出的内核参数。


### 4. 系统调用

### 5. Netlink













