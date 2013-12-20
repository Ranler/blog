layout: post
title: Linux Init
date: 2013-8-6
categories: Linux
---

init读取与系统有关的初始化文件，并将系统引导到一个状态。
init是普通的用户进程，以root权限运行。
init进程能够成为所有孤儿进程的父进程。

现在流行的两种init：

- SysVinit
- Systemd

### 终端登录和网络登录

1. 终端登录过程

init进程读文件`/etc/ttys`，针对每一个可登陆的**终端设备**（有限）`fork`一个子进程，
然后子进程`exec getty`。getty程序open终端设备阻塞等待用户登录，读取用户输入的用户名，
作为参数传给`exec login`程序继续读入用户密码，
等待验证成功后，设置用户权限和环境变量，启动shell。

2. 网络登录过程

init继承读取`/etc/rc`启动系统daemon，比如sshd。
sshd等待TCP/IP请求，然后针对每个请求打开一个**伪终端设备**，并`fork`一个子进程执行`exec login`进行登录。等待验证成功后，设置用户权限和环境变量，启动shell。

需要注意的是，shell的标准输入，标准输出和标准错误会连接到一个终端设备或伪终端设备上。

### SysVinit

SysVinit是比传统的init，现在在Redhat,Debian上还在使用着。
其中init的程序为：

``` sh
> which init
/sbin/init
> file /sbin/init
/sbin/init: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), stripped
```


相关配置文件：

``` sh
/etc/inittab           // check run level
/etc/rc*.d/*           // run level script
/etc/init.d/*          // daemon
```

### Systemd

Systemd是新式的init，兼容SysVinit，在Arch和Fedora上开始使用。
其init程序就软链接到systemd程序：

``` sh
> file -l /sbin/init
/sbin/init: symbolic link to `../usr/lib/systemd/systemd' 
```

相关配置文件：

``` sh
/etc/systemd/*
```



SysVinit和Systemd的命令对比：[SysVinit to Systemd Cheatsheet](http://fedoraproject.org/wiki/SysVinit_to_Systemd_Cheatsheet)

### Init in Solaris

``` sh
> which init
/usr/sbin/init
> file /usr/sbin/init 
/usr/sbin/init: ELF 32-bit MSB executable SPARC Version 1, dynamically linked, stripped
```




