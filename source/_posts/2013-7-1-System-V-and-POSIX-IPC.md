layout: post
title: System V IPC and POSIX IPC
date: 2013-7-1
categories: Linux
---

Both **System V IPC (XSI IPC)** and **POSIX IPC** including three different mechanisms for inter-processes communication:

- Message queues (消息队列): pass messages between processes
- Semaphores (信号量): synchronize for multiple processes by kernel
- Shared memory (共享存储): share region of memory

Summary of programming interfaces for System V IPC:

| Interface                 | Message queues | Semaphores  | Shared memory |
|---------------------------|----------------|-------------|---------------|
| Header file               | <sys/msg.h>    | <sys/sem.h> | <sys/shm.h>   |
| Associated data structure | msqid_ds       | semid_ds    | shmid_ds      |
| Create/open object        | msgget()       | semget()    | shmget()+shmat()|
| Close object              | (none)         | (none)      | shmdt()       |
| Control operations        | msgctl()       | semctl()    | shmctl()      |
| Performing IPC            | msgsnd()—write message, msgrcv()—read message | semop()—test/adjust semaphore | access memory in shared region |

A single system call (`ipc`(2) ) acts as the entry point to the kernel for all System V IPC operations.

Use command `ipcs -l` to show the built-in limit of System V IPC.

Summary of programming interfaces for POSIX IPC:

| Interface     | Message queues | Semaphores     | Shared memory |
|---------------|----------------|----------------|---------------|
| Header file   | <mqueue.h>     | <semaphore.h>  | <sys/mman.h>  |
| Object handle | mqd_t          | sem_t *        | int  (file descriptor) |
| Create/open   | mq_open()      | sem_open()     | shm_open() + mmap()    |
| Close         | mq_close()     | sem_close()    | munmap()               |
| Unlink        | mq_unlink()    | sem_unlink()   | shm_unlink()           |
| Perform IPC   | mq_send(), mq_receive() | sem_post(),  sem_wait(), sem_getvalue() | operate on locations in shared region |
| Miscellaneous operations | mq_setattr() —set attributes, mq_getattr() —get attributes, mq_notify()—request notification | sem_init()—initialize unnamed semaphore, sem_destroy()—destroy unnamed semaphore | (none) |


### Message queues

##### System V

For System V Message queues, It is recommended that Using PIPE/FIFO or UNIX domain sockets instead of Sysmtem V Message queues.

##### POSIX

POSIX message queues also have the following specific advantages over System V message queues:

- The message notification feature allows a (single) process to be asynchronously notified via a signal or the instantiation of a thread when a message arrives on a previously empty queue.
- On Linux (but not other UNIX implementations), POSIX message queues can be monitored using  poll(),  select() , and  epoll . System V message queues don’t pro-vide this feature.


### Semaphores

##### System V

Both Semaphores and File Lock are used for share resources between processes.
File Lock is simple than Semaphores, and Semaphores is a bit quicker than File Lock.

##### POSIX

POSIX semaphores have the following further advantages over System V semaphores:
- The POSIX semaphore interface is much simpler than the System V sema-phore interface. This simplicity is ac hieved without loss of functional power.
- POSIX named semaphores eliminate the initialization problem associated with
System V semaphores

### Shared memory

There are a number of different techniques for sharing memory regions between unrelated processes:

- System V shared memory
- shared file mappings
- POSIX shared memory objects

A number of points apply to all of these techniques:

- They provide fast IPC, and applications typically must use a semaphore (or other synchronization primitive) to synchronize access to the shared region.
- Once the shared memory region has be en mapped into the process’s virtual address space, it looks just like any other part of the process’s memory space.
- The system places the shared memory regions within the process virtual address space in a similar manner. We outlined th is placement while describing System V shared memory. The Linux-specific /proc/ PID /maps file lists infor-mation about all types of shared memory regions.
- Assuming that we don’t attempt to map a shared memory region at a fixed address, we should ensure that all refere nces to locations in the region are cal-culated as offsets (rather than pointers),  since the region may be located at dif-ferent virtual addresses within  different processes.

There are also a few notable differences between the techniques for shared memory:

The fact that the contents of a shared  file mapping are synchronized with the
- underlying mapped file means that the data stored in a shared memory region can persist across system restarts.
- System V and POSIX shared memory use different mechanisms to identify and refer to a shared memory object. System V uses its own scheme of keys and identifiers, which doesn’t fit with th e standard UNIX I/O model and requires separate system calls (e.g., shmctl()) and commands ( ipcs  and  ipcrm). By con-trast, POSIX shared memory employs names and file descriptors, and conse-quently shared memory objects can be examined and manipulated using a variety of existing UNIX system calls (e.g.,  fstat()  and fchmod() ).
- The size of a System V shared memory segment is fixed at the time of creation (via  shmget() ). By contrast, for a mapping backed by a file or by a POSIX shared memory object, we can use  ftruncate() to adjust the size of the underlying object, and then re-create the mapping using  munmap() and  mmap() (or the Linux-specific mremap() )
- Historically, System V shared memory  was more widely available than mmap() and POSIX shared memory, although most UNIX implementations now pro-vide all of these techniques.

### System V IPC vs POSIX IPC

##### TLPI

POSIX IPC has the following general advantages when compared to System V IPC:

- The POSIX IPC interface is simpler than the System V IPC interface.
- The POSIX IPC model—the use of names instead of keys, and the open, close , and  unlink functions—is more consistent with the traditional UNIX file model.
- POSIX IPC objects are reference counted. This simplifies object deletion,
because we can unlink a POSIX IPC object, knowing that it will be destroyed
only when all processes have closed it.

However, there is one notable advantage in favor of System V IPC: **portability**.

##### Stackoverflow

Question from Stackoverflow: [System V IPC vs POSIX IPC](http://stackoverflow.com/questions/4582968/system-v-ipc-vs-posix-ipc)

```
1. What are the differences between System V IPC and POSIX IPC ?
2. Why do we have two standards ?
3. How to decide which IPC functions to use ?
```

Answer 1:

```
Both have the same basic tools -- semaphores, shared memory and message queues. They offer a slightly different interface to those tools, but the basic concepts are the same. One notable difference is that POSIX offers some notification features for message queues that Sys V does not. (See mq_notify().)

Sys V IPC has been around for longer which has a couple of practical implications --

First, POSIX IPC is less widely implemented. I wrote a Python wrapper for POSIX IPC and its documentation lists what I know about POSIX IPC implementations on various platforms.

On all of the platforms listed in that documentation, Sys V IPC is completely implemented AFAIK, whereas you can see the POSIX IPC is not.

The second implication of their relative age is that POSIX IPC was designed after Sys V IPC had been used for a while. Therefore, the designers of the POSIX API were able to learn from the strengths and weaknesses of the Sys V API. As a result the POSIX API is simpler and easier to use IMO, and I recommend it over the Sys V API.

I should note that I've never run any performance tests to compare the two. I would think that the older API (Sys V) would have had more time to be performance tuned, but that's just speculation which is of course no substitute for real-world testing.

As to why there are two standards -- POSIX created their standard because they thought it was an improvement on the Sys V standard. But if everyone agreed that POSIX IPC is better, many many many programs still use Sys V IPC and it would take years to port them all to POSIX IPC. In practice, it would not be worth the effort so even if all new code used POSIX IPC as of tomorrow, Sys V IPC would stick around for many years.

We can't tell you which you should use without knowing a lot more about what you intend to do, but the answers you have here should give you enough information to decide on your own.
```

Answer 2:

```
1. I believe the major difference is that all POSIX IPC is thread-safe, while most SysV IPC is NOT.
2. Because of Unix wars. The Single UNIX specification (SUS), aka POSIX, was created to standardise interfaces on Unix-based systems.
3. You probably want POSIX. Depends exclusively on your requirements.

```

### Reference

- TLPI
