layout: post
title: Remote Process Call
date: 2013-4-19
categories: Linux
---

之前见到的RPC都是Remote Procedure Call，
比如Java的RMI(Remote Method Invocation)。
而最近的项目使用了Remote Process Call/远程程序调用，
也就是用主机A调用主机B上的程序，并返回执行结果。
这有些类似SSH的功能，只不过没有SSH的加密功能。
今天分析的都是这种RPC。

单一主机内的IPC(Inter-Process Communication)常常利用PIPE,FIFO机制实现。
而跨主机IPC都是通过网络来进行的。
因此，RPC作为IPC的一种应用，常常利用Socket机制来实现。
主机A上的请求进程使用Socket发送请求，
主机B上的服务进程使用Socket接收请求，执行请求，并使用Socket返回请求结果。

作为两个远程进程之间的通信，他们必需商定一种协议相互配合完成RPC。
这个协议就是对请求的封装，封装成双方都认识的数据结构，从而能了解对方的意图。
一个对远程程序调用的请求可以用以下数据结构简单表示：

```c
struct RemoteProcessCall {
	char cmd[256];       // 命令字符串，带参数
	char stdout[10240];  // 命令STDOUT输出
};
```

这几乎是一个最简单的RPC数据结构，
没有STDIN，STDOUT，STDERR部份信息，命令和参数也合在了一起，并有一个最大长度上限。
请求进程只需把需要执行的命令填入cmd部份，
服务进程在其主机上执行这个命令，并获得程序的STDOUT填入stdout数组，返回即可。
请求结构和返回结构不一定使用同一个数据结构。
可以使用两个数据结构，一个表示请求，一个表示返回，防止冗余，并映射到使用同一个ID号即可。

有了PRC数据结构, 下面考虑怎样在两个主机上发送和接收。
首先定义一个头文件包含使用的公共数据结构：

```c
#ifndef _RPC_H_
#define _PRC_H_

struct RemoteProcessCall {
  char cmd[256];
  char stdout[10240];
};

#define RPCHOST "127.0.0.1"
#define RPCPORT 8081

#endif
```


然后考虑较简单的请求进程：

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include "rpc.h"

int main(int argc, char *argv[])
{
  int rpc_fd;
  struct sockaddr_in rpc_addr;

  struct RemoteProcessCall rpc;
  ssize_t nbytes;
  ssize_t recvd_nbytes;
  
  memset(&rpc_addr, 0, sizeof(rpc_addr));
  rpc_addr.sin_family = AF_INET;
  rpc_addr.sin_addr.s_addr = inet_addr(RPCHOST);
  rpc_addr.sin_port = htons(RPCPORT);

  if((rpc_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
    perror("create socket error!");
    return 1;
  }

  if(connect(rpc_fd, (struct sockaddr*)&rpc_addr, sizeof(struct sockaddr)) < 0)   {
    perror("connect error!");
    return 2;
  }

  // send rpc call
  memcpy(rpc.cmd, argv[1], strlen(argv[1])+1);
  if((nbytes = send(rpc_fd, &rpc, sizeof(rpc), 0)) < sizeof(rpc)) {
    perror("send error!");
    return 3;
  }
  
  // recieve rpc call
  recvd_nbytes = 0;
  while((nbytes=recv(rpc_fd, (char*)&rpc+recvd_nbytes, sizeof(rpc), 0)) > 0) {
    recvd_nbytes += nbytes;
    if (recvd_nbytes == sizeof(rpc)) break;
  }
  if (recvd_nbytes != sizeof(rpc)) {
    perror("recv call bad!");
    close(rpc_fd);
    return 4;
  }

  // output result
  printf("%s\n", rpc.stdout);

  close(rpc_fd);
  return 0;
}
```

基本就是单一进程socket访问的流程，创建socket，connect连接服务端，
send发送构造好的数据包，
recv阻塞等待调用完成，然后输出到终端。

服务端复杂一些，首先创建服务端Socket侦听程序，
一旦有请求过来，fork一个子进程进入run_rpc处理RPC请求。
子进程使用popen系统调用执行命令，这样可以获取程序的输出，但是不能获取返回值。
其它执行命令的方案可以是system()，可以获取程序的返回值但不能获取输出。
或者是再fork()子进程并exec()去执行命令，这样可以进行更详细的控制。
popen执行命令之后，获取输出填入rpc call并发回。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <sys/wait.h>
#include "rpc.h"

void run_rpc(int fd) {
  struct RemoteProcessCall call;
  FILE* clfd;
  ssize_t nbytes;
  int recvd_nbytes;

  recvd_nbytes = 0;
  while((nbytes=recv(fd, (char*)&call+recvd_nbytes, sizeof(call), 0)) > 0) {
    recvd_nbytes += nbytes;
    if (recvd_nbytes == sizeof(call)) break;
  }
  if (recvd_nbytes != sizeof(call)) {
    perror("recv call bad!");
    close(fd);
    exit(-1);    
  }
  printf("recv command: %s\n", call.cmd);

  clfd = popen(call.cmd, "r");
  if((nbytes=fread(call.stdout, 1, sizeof(call.stdout), clfd)) < 0) {
    perror("get stdout error!");
    exit(-1);
  }
  pclose(clfd);
  printf("run command over,  stdout size: %d\n", nbytes); // TODO nbytes

  if((nbytes=send(fd, &call, sizeof(call), 0)) != sizeof(call)) {
    perror("send call bad!");
    close(fd);
    exit(-1);
  }
  printf("send result: %s\n", call.cmd);

  close(fd);
  exit(0);
}

void sig_child(int signo)
{
    int status;
    wait(&status);
    //printf("process exit status %d\n", WEXITSTATUS(status));
}

int main(int argc, char *argv[])
{
  int server_fd, client_fd;
  struct sockaddr_in server_addr;
  int pid;

  if((server_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
    perror("create socket error!");
    return 1;
  }
  
  server_addr.sin_family = AF_INET;
  server_addr.sin_port=htons(RPCPORT);
  server_addr.sin_addr.s_addr = INADDR_ANY;
  bzero(&(server_addr.sin_zero), 8);

  if(bind(server_fd, (struct sockaddr*)&server_addr, sizeof(struct sockaddr)) < 0) {
    perror("bind error!");
    return 2;
  }

  if(listen(server_fd, 10) < 0) {
    perror("listen error!");
    return 3;
  }
  
  signal(SIGCHLD, sig_child);
  for(;;) {
    if((client_fd = accept(server_fd, NULL, NULL)) < 0) {
      perror("accept error!");
      continue;
    }
    
    if((pid = fork()) < 0) {
      perror("fork error!");
      close(client_fd);
      close(server_fd);
      return 4;
    }
    if(pid == 0) {    // child
      close(server_fd);
      run_rpc(client_fd);
    }
    // father
    close(client_fd);
  }
  return 0;
}
```

执行的效果如下，服务端：

```sh
$./server                        
recv command: ps
run command over,  stdout size: 145
send result: ps
recv command: ps aux
run command over,  stdout size: 6175
send result: ps aux
```

客户端：

```sh
$./client "ps"
  PID TTY          TIME CMD
  269 pts/1    00:00:00 zsh
  480 pts/1    00:00:00 server
  492 pts/1    00:00:00 server
  493 pts/1    00:00:00 ps

$./client "ps aux"
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  1.0   4964  2652 ?        Ss   08:59   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    08:59   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    08:59   0:00 [ksoftirqd/0]
root         4  0.0  0.0      0     0 ?        S    08:59   0:00 [kworker/0:0]
root         5  0.0  0.0      0     0 ?        S<   08:59   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S    08:59   0:00 [kworker/u:0]
root         7  0.0  0.0      0     0 ?        S<   08:59   0:00 [kworker/u:0H]
...
```


更多可扩展的问题：

- 服务端转为Daemon进程
- 网络字节序的调整
- stderr的获取
- 网络传输字节的加密

