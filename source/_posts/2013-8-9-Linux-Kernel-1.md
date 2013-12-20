layout: post
title: Linux Kernel Quickstart
date: 2013-8-9
categories: Linux
---

Linux is a preemptive mutlitasking operating system.

Typical components of a kernel are:

- interrupt handles to service interrupt requests(abstract of CPU)
- a scheduler to share processor time among multiple processes(multitasking)
- a memory management to manage process address spaces(virtual memory)
- services such as networking and IPC(communication between Processes)


Each processor is doing exactly one of three things at any given moment:

- In user-space, executing user code in process
- In kernel-space, in process context, executing on behalf of a specific process
- In kernel-space, in interrupt context, not associated with a process, handing an interrupt



- monolithic kernel
- microkernel


### Process

process provide two virtualizations:

- virtualized processor
- virtual memory

**Task list** is a cirular doubly linked list.
Each element in the task list is a process descriptor of the type `struct task_struct`.
The `task_struct` is at around 1.7KB on a 32-bit machine.
The `task_struct` structure is allocated via the **slab allocator** to provide object reuse and cache coloring.

The max value of process may increase via `/proc/sys/kernel/pid_max`.

There are five flags for process states:

- TASK_RUNNING: 执行态，就绪态
- TASK_INTERRUPTIBLE: 阻塞态，响应signal
- TASK_UNINTERRUPTIBLE: 阻塞态，不响应signal
- __TASK_TRACED: 进程被别的进程traced，如gdb, pstrace
- __TASK_STOPED: 进程收到SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU信号，或debug时收到任意信号


When creating a process, by COW technology, the only overhead by `fork()` is duplication of the parent's page tables and new a PID.
The only benifit to `vfork()` is not copying the parent page tables entries.
`fork()` implemented via the `clone()`.

Threads are also created via `clone()` of different flags,
with sharing the address space, filesystem resources, file descriptors and signal handlers.

```
clone(SIGCHLD, 0);                                             // normal fork
clone(CLONE_VFORK | CLONE_VM | SIGCHLD, 0);                    // vfork
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);   // create thread
```

Compare with normal theads ,kernel threads do not have an adress space.
The notably kernel threads are `flush` and `ksoftiqd`.


### Process Scheduling

- cooperative multitasking
- preemptive multitasking

Two separate priority range:

- nice value: -20 ~ +19
- real-time priority: 0~99, only for real-time processes

**Completely Fair Scheduler(CFS)** uses a **red-black tree** to manage the list of runable processes 
and efficiently find the process with the smallest **vruntime**.
When process is sleeping, remove it from red-black tree, put it into **wait queue** to wait an event , and call `schedule()` to select a new process to excute.

`schedule()` call `context_switch()` to do two basic jobs:

- `switch_mm()` to switch the virtual memory mapping
- `switch_to()` to switch stack information and the processor registers


- User Preemption can occur
  - When returning to user-space from a system call
  - When returning to user-space from an iterrupt handler
- Kernel Preemption can occur
  - When an interrupt handler exists, before returning to kernel-space
  - When kernel code becomes preemptible again
  - If a task in the kernel explicitly calls `schedule()`
  - If a task in the kernel blocks (which results in a call to `schedule()`)

Linux provide soft real-time scheduling policies, `SCHED_FIFO` and `SCHED_RR`.
The not real-time scheduling policy is `SCHED_NORMAL`.
These real-time policies are managed not by CFS.


### System Calls

- provide an abstracted hardware interface for userspace
- ensure system security and stability
- provide virtualized system to processes

Linux's system calls is POSIX and SUSv3 compliant.

User-space process use a software interrupt and trap into kernel-space to execute a system call.
The system call handler(`system_call()`) act as exception handler to be executed when an exception occur and system switch to kernel mode.
The defined software interrupt on x86 is 128(`int $0x80`).

Recently, x86 processors added a feature known as `sysenter` to provide a faster, more specialized way of trapping into a kernel to execute a system call than using the `int` interrupt instruction.

Each system call is assigned a unique syscall number.
Before the trap into system call, user-space process set syscall number into `eax`register.
`system_call()` check this value and call the specified system call in system call table.
The parameters also store in registers(`ebx,ecx,edx,esi,edi`) and the return value written into `eax` register in easiest time.

The kernel provides two methods `copy_to_user()` and `copy_from_user()` for performing the requisite checks and the desired copy to and from user-space.


### Kernel Data Structures

- Linked Lists
  - Singly and Doubly Linked Lists
  - Circular Linked Lists
- Queues
- Maps
- Binary trees

#### Circular Doubly Linked List

Linux embed a linked list node in the structure instead of turning the structure into a linked list.

```.c
struct list_head {
	struct list_head *next;
	struct list_head *prev;
};
```

```.c
struct fox {
	long data;
	struct list_head list;
};

struct fox *f = kmalloc(sizeof(*f), GFP_KERNEL);
INIT_LIST_HEAD(&f->list);
```

O(1) operations:

```.c
list_add(struct list_head *new, struct list_head *head);
list_add_tail(struct list_head *new, struct list_head *head);
list_del(struct list_head *entry);
list_del_init(struct list_head *entry);
list_move(struct list_head *list, struct list_head *head);
list_move_tail(struct list_head *list, struct list_head *head);
list_empty(struct list_head *head);
list_splice(struct list_head *list, struct list_head *head);
list_splice_init(struct list_head *list, struct list_head *head);
```

O(n) operations:

```.c
struct list_head *p;
struct fox *f;

list_for_each(p, &f_list) {
	f = list_entry(p, struct fox, list);
	...
}

list_for_each_entry(f, &f_list, list) {
	...
}

list_for_each_entry_reverse(f, &f_list, list) {
	...
}

list_for_each_entry_safe(f, next, &f_list, list) {
	// safety remove f from f_list
	...
}

```

#### Queue

Linux kernel provide `kfifo` as queue and some API for it.

#### Map

Map also known as associative array.

- Map: UID-to-pointer mapping
- Binary Search Tree
- Self-Balancing Binary Search Tree
- Red-Black Tree (semi-balanced)


### Interrupts and Interrupt Handlers

- interrupt: asynchronous interrupts generated by hardware
- exception: synchronous interrupts generated by processor

Both interrupts and exceptions have similar handlers.
These handlers run in a special context called **interrupt context** in which code executing is unable to block.

The processing of interrupts is split into two parts:

- top half: an interrupt handler execute quickly
- bottom half: perform a large amount of work

Interrupt values are often called interrupt request (IRQ) lines.
Each IRQ line is assignd a numeric value.
For some devices, this value is typically hard-coded.
For most devices, it is probed or otherwise determined programmatically and dynamically.

`/proc/interrupts` file list all interrupts in running Linux.


### Kernel Synchronization

Locks are **advisory** and **voluntary**.

Almost all processors implement an atomic **test** and **set** instruction that tests the value of an integer ans sets it to a new value only if it is zero.
On the popular x86 architecture, locks are implemented using such a similar instruction called **compare** and **exchange**.

- interrupt-safe
- SMP-safe      `CONFIG_SMP`
- prempt-safe   `CONFIG_PREEMPT`


Big Fat Rule: lock data, not code.


#### 1. atomic integer

```
atomic_t v;
atomic_t u = ATOMIC_INIT(0);

atomic_set(&v, 4);
atomic_add(2, &v);
atomic_inc(&v);
printf("%d\n", atomic_read(%v));       // atomic_t -> int
...                                    // more functions in <asm/atomic.h>
```

There also has `atomic64_t` and functions for 64-bit architectures.

#### 2. atomic bitwise

Atomic bitwise operations on generic memory addresses in <asm/bitops.h>

#### 3. spin lock 自旋锁

Spin lock is a lightweight single-holder lock that should be held for short durations.(keep runing, not sleep)

```.c
// <linux/spinlock.h>, <linux/spinlock.h>
DEFINE_SPINLOCK(mr_lock);
spin_lock(&mr_lock);
spin_unlock(&mr_lock);
...
```

WARING: Spin lock are **not recursive**.
Spin lock can be used in interrupt handlers, where semaphores cannot be used because they sleep.
It must disable interrupts to prevent deadlock.


#### 4. reader-writer spin lock

Reader/Writer, Consumer/Producer, Shared/Exclusive, Concurrent/Exclusive locks

```.c
DEFINE_RWCLOCK(mr_rwlock);

read_lock(&mr_rwlock);
// critical section
read_unlock(&mr_rwlock);

write_lock(&mr_rwlock);
// critical section
write_unlock(&mr_rwlock); 
```

#### 5. semaphore

Semaphores are sleeping locks and have much greater overhead than spin locks.
When a task attempt to acquire a semaphore that is unavaiable, the semaphore places the task onto a wait queue and puts the task to sleep.
Semaphores are well suited to locks that are held for a long time and must be obtained only in process context(not in interrupt context).
You cannot hold a spin lock while you acquire a semaphore.

Number of simultaneous lock holders:

- 1: binary semaphore, mutex
- bigger than 1: counting semaphore

- `P()`: Proberen, `down()`
- `V()`: Verhogen, `up()`

```.c
// <asm/semaphore.h>
struct semaphore name;
sema_init(&name, count);      // counting semaphore
static DECLARE_MUTEX(name);   // binary semaphore

if (down_interruptible(&name)) {
	// signal received, semaphore not acquired
}

// critical region

up(&name)
```

Semaphores also have reader-writer semaphores.

#### 6. mutex

There is a new sleeping lock named mutex which is different with binary semphore.
This mutex is simpler and lighter than binary semphore.

Other Locks:

- Completion Variables
- The Kernel Lock
- Sequential Locks


#### Ordering and Barries

Both the compiler(static) and the processor(dynamical) can **reorder** reads and writes for performance reasons.
**Barrier instruction** ensure that no loads/stores prior to the instruction will be reordered to after the instruction
and no loads/stores after the instruction will be reordered to before the instruction.

- `rmb()`/`smp_rmb()`: read memory barrier
- `wmb()`/`smp_wmb()`: write memory barrier(x86 CPU does not perform out-of-order stroes, so this does nothing)
- `mb()`/`smp_mb()`: read and write barrier
- `read_barrier_depends()`: read barrier only for loads on which subsequent loads depend
- `barrier()`: prevent compiler reorder


### Timers and Time Management

#### time interrupt

Works executed periodically on every timer interrupt:

- Updating the system uptime and time of day
- Updating resouce usage and processor time statistics
- Balancing the scheduler runquenes on an SMP system
- Running any dynamic timers that have expired

The tick rate has a frequency of `HZ` hertz and a period of 1/HZ seconds.
By default, the x86 architecture defines HZ to be 100.
This means that the time interrupt handler runs every 10ms.

```.c
// <include/asm-generic/param.h>
   8 # define USER_HZ        100             /* some user interfaces are */
```

The golbal variable `jiffies` holds the number of ticks that occurred since the system booted.

The timer interrupt handler also into two pieces: an arch-dependent and an arch-independent routime(`tick_periodic()`).
`tick_periodic()` will call `schedular_tick()` to reschedular processes and balancing unquenes on SMP.


#### timer

Timers/Dynamic Timers/Kernel Timers offer process to delay exection until a late time.

```.c
struct timer_list my_timer;
my_timer.expires = jiffies + delay;
my_timer.data = 0;
my_timer.function = my_function;

add_timer(&my_timer);
mod_timer(&my_timer, jiffies + new_delay);
del_timer(&my_timer);
```

#### Delaying Exection

Delaying exection for busy looping and small delays.


### Memory Management

- kernel physical address: Physical 
- kernel logical address: Segment + Offset
- kernel virtual/linear address: MMU

logical address and virtual address are same in kernel's address space?

#### Physical Pages

The kernel treats **physical pages** as the basic unit of memory management.
The memory management unit(MMU) also typically deals in pages.
Most 32-bit architectures have 4KB pages, whereas most 64-bit architecture have 8KB pages.
The kernel repesents **every physical page** with a `strcut page` structure(<linux/mm_types.h>).
Pages's owners include:

- user-space processes
- dynamically allocated kernel data
- static kernel code
- page cache

#### Zones

The kernel divides physical pages into different zones(logical groupings):

- ZONE_DMA: This zone contains pages that can undergo DMA
- ZONE_DMA32: same as ZONE_DMA, but accessible only by 32-bit devices
- ZONE_NORMAL: normal pages
- ZONE_HIGHMEM: not permanently mappred into the kernel's address space. 

Some architectures can physically addressing larger acmounts of memory than they can virtually address.
So, some memory(ZONE_HIGHMEM) is not permanently mapped into the kernel address space.
Pages in ZONE_HIGHMEM must be mapped into ZONE_NORMAL.

Physical memory address:

| Zone         | Description                 | x86-32    | x86-64     |
|--------------|-----------------------------|-----------|------------|
| ZONE_DMA     | DMA-able pages              | < 16MB    | < 1GB      |
| ZONE_NORMAL  | Normally addresssable pages | 16-896MB  | 1GB - 64GB |
| ZONE_HIGHMEM | Dynamically mapped pages    | > 896MB   | None       |

Virtual memory address:

| virutal memory address | x86-32     | x86-64     |
| User space             | 0G-3G      | 0G- >512G  |
| Kernel space           | 3G-4G      | >512G      |


#### Allocate and Free Pages and Bytes

Low-level page function:

```.c
// allocate
struct page * alloc_page(gtp_t gfp_mask, unsigned int order);     // get contiguous physical pages, number of 2^order pages
void * page_address(struct page *page);                           // return logical address
unsigned long __get_free_pages(gfp_t gfp_mask, unsigned int order);
unsigned long get_zeroed_page(unsigned int gfp_mask);

// free
void __free_pages(struct page *page, unsigned int order);
void free_pages(unsigned long addr, unsigned int order);
void free_page(unsigned long addr);
```

Byte-sized allocation:

```.c
// <linux/slab.h>
void * kmalloc(size_t size, gfp_t flags);   // at least size bytes physically contiguous region, return logical address
void kfree(const void *ptr);

// <linux/vmalloc.h>
void * vmalloc(unsigned long size);   // virtually contiguous region, return virtual address
void vfree(const void *addr);
```

| allocate | physically contiguous | virtually contigous | environment                                  |
|---------|-----------------------|----------------------|----------------------------------------------|
| kmalloc | YES                   | YES                  | used by hardware(DMA) or has a better performance |
| vmalloc | NO                    | YES                  | large regions of memory                      |

#### Slab Layer

**Free List** is often introduced to allocate and deallocate data.
But there exists no global control in linux kernel and the kernel has no understanding of the random free lists at all.
So the kernel provides the **slab layer/allocator** as a generic data **structure-caching** layer(分配特定大小的对象).

Different types of object (such as `task_struct`, `inode`) have their own groups called **caches**.
Each cache has many **slabs**.
Each slab is in one the three state: full, partial or empty.
Each slab is composed of one or more physical contiguous pages.
Each slab has many target **objects**.

Each cache `struct kmem_cache` has three lists:

- `slabs_full`: a full slab list
- `slabs_partial`: a partial slab list
- `slabs_empty`: a empty slab list 

When `inode` need to allocate, it is get `inode_cachep` cache and find a slab in partial slab list, return a free object.

`kmalloc()` also use slab layer to allocate and free byte-based memory.
Slab layer use `__get_free_pages()` or  `alloc_page()` to allocate and free page-based memory.

```
kmalloc()   ===>   Slab Layer   ===>  __get_free_pages()
```

#### Kernel Stack and Interrupt Stack

There are one or two(default) pages for **kernel stack** per process.
This is usually 8KB for 32bit architecture and 18KB for 64bit architecture by default.

**Interrupt stacks** provide a single page per processor stack used for interrupt handlers.

#### High Memory Mapping

Pages physical address in ZONE_HIGHMEM(over 896MB) must be mapped into the kernel's logical address(3GB-4GB on x86).

```
void *kmap(struct page *page);     // physical address => logical address, only work in process context
void kunmap(struct page *page);    // free logical address
void *kmap_atomic(struct page *page, enum km_type type); // temporary mappings
```

### The Virtual Filesystem

Together, these two modules provide the abstractions, interfaces, and glue that allow user-space programs to issue generic system calls to access files via a uniform naming policy on any filesystem, which itself exists on any storage medium.

- VFS
- block I/O layer

VFS contains these typies of objects:

- `struct file`: an opened file and directory file
- `struct entry`: a component of a path for link inode
- `struct inode`: an special index node for file metadata such as access permissions, size, owner, creation time and so on
- `struct super_block`: an special filesystem metadata

Operations objects on these typies of objects that target filesystem should be implemented :

- `struct super_operations`: such as `write_inode()`, `sync_fs()`
- `struct inode_operations`: such as `create()`, `link()`
- `struct dentry_operations`: such as `d_compare()`, `d_delete()`
- `struct file_operations`: such as `read()`, `write()`

```
file1 -------> entry1 --------> inode -------> disk media
file2 ---|               |
file3 -------> entry2 ---|
```

### The Block I/O Layer

Device types:

| Type             | access        | devices                                          | usage                              |
|------------------|---------------|--------------------------------------------------|------------------------------------|
| block device     | randomly      | hard disk, Blu-ray reader, flash                 | generately mounted as a filesystem |
| character device | sequentially  | serial ports, keyboard, pseudo-devices, printers | directly through deive node        |
| ethernet device  | sequentially  | network device                                   | socket                             |


The smallest addressable unit on a block device is a **sector**(扇区).
The block is an abstraction of the filesystem which is operated on.
Although the physical device is addressable at the sector level,
the kernel performs all disk operations in terms of blocks.

| name                                  |    common size  |  type    |
|---------------------------------------|-----------------|----------|
| memory page                           | 4KB/8KB         | physical |
| block (filesystem blocks/IO blocks)   | 512B, 1KB, 4KB  | logical  |
| sector(hard sectors/device blocks)    | 512B            | physical |

A single page can hold one or more blocks in memory.
Each block is stored in a **buffer** which also associated with a descriptor to store control information.
This descriptor is called `struct buffer_head`.

In the 2.6 kernel, much more work has gone into making the kernel work directly with pages and address spaces instead of buffers.
The new `struct bio` represents block I/O operations that in flight as a list of segment `struct bio_vec`.

- `struct buffer_head`: represent a single buffer
- `struct bio`: represent one or more pages, both normal page I/O and direct I/O, much more lightweight


#### I/O Schedulers

Merging and sorting I/O request to minimize disk seeks and ensure optimum disk perfrmance
is two primary actions of I/O schedulars.

- The Deadline I/O Schedular
- The Anticipatory I/O Schedular
- The Complete Fair Queuing I/O Schedular (default)
- The Noop I/O Schedular


### The Process Address Space

Memory areas/sections for process address space(`/proc/PID/maps`):

- text, data, bss, stack section
- additional text, data and bss for each shared library
- memory mapped files
- shared memory segments
- anonymous memory mappings, such as those associated with `malloc()`. Newer versions of glibc implement `malloc()` via `mmap()`, in addition to `brk()`.

A process's address space is represented with a data struct called **memory descriptor** `struct mm_struct`.
Every section address is assigned in this struct.
Two threads are shared this struct object.

A section in process's address space is represented with a data struct called **virtual memory areas(VMA)** `struct vm_area_struct`. The operations on this struct are defined in `struct vm_operations_struct`.

The kernel threads do not have `mm_struct`.
So it's prcoess's `mm` field is NULL and it use previous process's page tables because kernel threads do not access user-space memory.


#### mmap

The `do_mapp()` function is used by the kernel to create a new linear address interval.
A new VMA will be created if the new interval and exist intervals can not be merged.
Two mapping types:

- anonymous mapping
- file-backed mapping

The `do_mmap()` functionality is exported to user-space via the `mmap()` system call.
New interface `mmap2()` replace `mmap()` to offer larger files to be mapped.

#### Page Tables

In Linux, the page tables consist of three levels:

- page global directory(PGD)
- page middle directory(PMD)
- simple the page table(PTE)

```
struct mm_struct.pgd  ===>  PGD ===> PMD ===> PTE ===> struct page ===> physical page
```

Most CPU implement a **translation lookaside buffer(TLB)** which acts as a hardware cache of virtual-to-physical mapping.


### The Page Cache and Page Writeback

Two level caches based on **temporal locality** principle:

- RAM pages cache in CPU
- disk blocks cache in RAM

A disk cache is called **page cache** in Linux.
**Two-list strategy**, a modified version of **least recently used(LRU)** algorithm, is a strategy in Linux for page cache.
Linux keeps two list(queue):

- active list: hot pages, get from inactive list
- inactive list: available for cache eviction

If the active list become much larger than the inactive list, 
items from the active list's head are moved back to the inactive list, 
making them available for eviction.


### Portability

On all current architectures in Linux:

- char: 1 byte, but on most architectures char is signed by default, on other machines char is unsigned by default(ARM)
- short: 16 bits
- int: 32 bits
- long: 32 bits(x86-32) or 64bits(x86-64)
- long long: 64bits



