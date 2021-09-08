基础api
===

[TOC]

## socket地址api

### 主机字节序和网络字节序

```c++
#include <netinet/in.h>//头文件
unsigned long int htonl (unsigned long int hostlong);//长整型主机到网络，用于ip转换
unsigned short int htons(unsigned short int hostshort);//短整型主机到网络，用于port转换
unsigned long int ntohl(unsigned long int netlong);//长整型网络到主机，用于ip转换
unsigned short int ntohs(unsigned short int netshort);//短整型网络到主机，用于port转换
```

***

### 专用socket地址

使用时直接强制转化为sockaddr

```c++
#include <sys/socket.h>
#include <netinet/in/h>
struct sockaddr_in//ipv4专用
{
    sa_family_t sin_family;//地址族，AE_INET
    u_int16_t sin_port;//端口号，要是网络字节序格式
    struct in_addr sin_addr;//ipv4地址
}
struct in_addr
{
    u_int32_t s_addr;//ipv4地址
}
```

***

### IP地址转换

```c++
#include <arpa/inet.h>
in_addr_t inet_addr(const char* strptr);//十进制字符串表示的IPv4地址转换为网络字节序
int inet_aton(const char*cp,struct in_addr* inp);//同上，结果存储在inp中，成功返回1，失败返回0
char* inet_ntoa(struct in_addr in);//网络字节序转化为十进制字符串表示的IPv4地址

/*
*@param af 协议族，ipv4为AF_INET，ipv6为AF_INET6
*@param src 源数据地址
*@param src 要存放结果的地址
*@param cnt 指定目标存储单元大小，IPv4为16，IPv6为46
*/
int inet_pton(int af,const char*src,void dst);//用于字符串IP转换到网络字节序整数表示的ip地址，支持IPv6，成功返回1，失败返回0
const char* inet_ntop(int af,const void* src,char *dst,socklen_t cnt);//用于网络字节序证书到字符串IP，支持IPv6，成功返回目标单元地址，失败返回NULL
```

***

### 创建socket

```c++
#include <sys/types.h>
#include <sys/socket.h>
/*
*@param domain 表示使用的协议族，对于TCP/IP而言，应该设置为PF_INET或PF_INET，对于UNIX本地而言，设置为PF_UNIX(用于进程间通信)
*@param type 表示服务类型，通常用SOCK_STREAM数据流服务(TCP)和SOCK_UGRAM数据报服务(UDP)，新版本内核可以添加相与的SOCK_NONBLOCK(表示非阻塞)和SOCK_CLOEXEC(表示子进程关闭socket)，也可以用fcntl设置
*@param protocol 具体协议，通常为0
*@return 成功时返回一个文件描述符，失败返回-1并设置errno
*/
int socket(int domain,int type,int protocol);
```

***

### 命名socket

将一个socket与socket地址绑定成为给其命名，客户端不需要

```c++
#include <sys/types.h>
#include <sys/socket.h>
/*
*@param sockfd socket文件描述符
*@param my_addr sockt具体地址
*@param addrlen socket地址长度
*@return 成功返回0.失败返回-1并设置errno
*/
int bind(int sockfd,const struct sockaddr* my_addr,socklen_t addrlen)
```

***

### 监听socket

用于创建一个监听队列来存放待处理的客户连接

```c++
#include <sys/socket.h>
/*
*param sockfd socket文件描述符
*param backlog 内核监听队列最大长度
*return 成功返回0，失败返回-1并设置errno
*/
int listen(int sockfd,int backlog);
```

***

### 接受连接

从listen监听队列中接受一个连接

```c++
#include <sys/socket.h>
#include <sys/types.h>
int accept(int sockfd,struct sockaddr* addr,socklen_t *addrlen);
/*
*@return 成功时返回socket来唯一标识该连接，失败时返回-1并设置errno
*@param addr 用于存储客户地址
*/
```

***

### 发起连接

```c++
#include <sys/socket.h>
#include <sys/types.h>
int connect(int sockfd,const struct sockaddr *serv_addr,socklen_t addrlen);
/*
*@return 成功时返回socket来唯一标识该连接，失败时返回-1并设置errno
*@param serv_addr 用于客户端地址
*/
```

***

### 关闭连接

```c++
#include <unistd.h>
int close(int fd);//用于文件描述符引用计数减一
/*该函数只能将引用计数减1，只有当引用计数为0的时候，才真正关闭连接。多进程程序里一次fork将使得socket的引用计数加1.必须在父进程和子进程都调用close才真正关闭。
```

```c++
#include <sys/socket.h>
int shutdown(int sockfd,int howto);//立即终止连接
/*
SHUT_RD:关闭sockfd读的功能
SHUT_WR:关闭sockfd写的功能
SHUT_RDWR:同时关闭读和写
*@return 0表示成功，-1表示失败并设置errno
*/
```

### 数据读写

