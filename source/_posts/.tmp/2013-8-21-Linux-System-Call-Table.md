---
layout: default
title: Linux System Call Table
---

Relatived files for Linux system call table in Linux 3.9:

- `arch/sh/include/uapi/asm/unistd_32.h`
- `arch/x86/syscalls/syscall_64.tbl`
- `root/include/linux/syscalls.h`
- `include/uapi/asm-generic/unistd.h`

For example `sys_read`:

```
// arch/x86/syscalls/syscall_64.tbl
# <number> <abi> <name> <entry point>
0       common  read                    sys_read
```

```.c
// root/include/linux/syscalls.h
asmlinkage long sys_read(unsigned int fd, char __user *buf, size_t count);
```



