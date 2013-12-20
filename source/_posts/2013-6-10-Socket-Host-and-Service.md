layout: post
title: Socket Host and Service
date: 2013-6-10
categories: Linux
---

一条Socket通过一个四元组{Remote IP, Remote Port, Local IP, Local Port}来标识。
其中IP和Port都是数字，不利于人们记忆。所以要把它们映射成容易记忆的字符名：

- IP => 主机名
- Port => 服务名

因此Socket接口必需支持两种映射。

### IP <==> 主机名

Socket中描述主机名的结构是：

```c
struct hostent {
    char  *h_name;              /* Official (canonical) name of host */
    char **h_aliases;           /* NULL-terminated array of pointers
                                   to alias strings */
    int    h_addrtype;          /* Address type (AF_INET or AF_INET6) */
    int    h_length;            /* Length (in bytes) of addresses pointed
                                   to by h_addr_list (4 bytes for AF_INET,
                                   16 bytes for AF_INET6) */
    char **h_addr_list;         /* NULL-terminated array of pointers to
                                   host IP addresses (in_addr or in6_addr
                                   structures) in network byte order */
};
```

获得目标的`struct hostent`函数是：

```c
// 主机名 => IP
#include <netdb.h>
extern int h_errno;
struct hostent *gethostbyname(const char *name);

// IP => 主机名
#include <sys/socket.h>       /* for AF_INET */
struct hostent *gethostbyaddr(const void *addr, socklen_t len, int type);
```

通过查看函数接口可以看出，使用目标服务名或者目标IP地址可以获得其`struct hostent`。
函数首先查询本地的静态主机文件(/etc/hosts)，如果没有取到则使用DNS协议访问/etc/resolv.conf中配置的名字服务器，查询目标主机信息。这些过程对于函数使用者是透明的。


### Port <==> 服务名

描述服务名的结构是：

```c
struct servent {
    char  *s_name;          /* Official service name */
    char **s_aliases;       /* Pointers to aliases (NULL-terminated) */
    int    s_port;          /* Port number (in network byte order) */
    char  *s_proto;         /* Protocol (TCP/UDP)*/
};
```

相关的函数是：

```c
#include <netdb.h>
// 服务名 => Port
struct servent *getservbyname(const char *name, const char *proto);
// Port => 服务名
struct servent *getservbyport(int port, const char *proto);
```

这些函数仅仅通过查询本地的(/etc/services文件)获得相应服务名。

### 通用型

以上函数在早期都不支持IPv6, 并且是**不可重入的**，因为它们都返回指向同一个静态结构的指针(还有错误号变量的问题)。
下面两个函数更早地支持IPv6和可重入。(P294)

`getaddrinfo`函数能够处理主机名到IP地址的转换：

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
int getaddrinfo(const char *node, const char *service,
	            const struct addrinfo *hints,
				struct addrinfo **res);
void freeaddrinfo(struct addrinfo *res);
```
`getnameinfo`能够处理服务名到端口的转换：

```c
#include <sys/socket.h>
#include <netdb.h>
int getnameinfo(const struct sockaddr *sa, socklen_t salen,
                char *host, size_t hostlen,
                char *serv, size_t servlen, int flags);
```





