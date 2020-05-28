# /

## 数据库

### 关系型数据库

- MySQL

	- 锁

		- 行锁
		- 表锁
		- MDL

	- 事务

		- 读未提交
		- 读已提交
		- 可重复读
		- 串行化

	- Log

		- Redo Log
		- Bin Log

	- ReadView
	- 优化

- sqlite3
- Oracle
- PostgreSQL

### NoSQL

- redis

	- 类型

		- list
		- string
		- set
		- zset

- mongoDB

## 消息队列

### RocketMQ

### Kafka

### RabbitMQ

### ActiveMQ

### ZeroMQ

## 算法

### 排序

- 快速排序
- 冒泡排序
- 堆排序
- 归并排序

### 查找

- 二分查找

## 数据结构

### 栈

### 堆

### 树

- 二叉树
- B+树

### 图

### 队列

### 哈希

### 线性表

## 设计模式

## 网络

### OSI参考模型

- 应用层
- 表示层
- 会话层
- 传输层
- 网络层
- 数据链路层
- 物理层

### TCP / IP

- TCP

  又叫字节流套接字(Stream Socket)
  
  SOCK_STREAM
  
  面向连接的数据流协议
  
  高可靠性

	- 协议栈

		- 应用层
		- 传输层
		- 网际层
		- 网络接口层

	- 函数

		- socket

		  int socket(int domain, int type, int protocol)
		  
		  domain：
		  PF_INET：IPv4
		  PF_INET6：IPv6
		  PF_LOCAL：本地
		  
		  type：
		  SOCK_STREAM：表示字节流，对应TCP
		  SOCK_DGRAM：表示数据报，对应UDP
		  SOCK_RAW：表示原始套接字

		- bind

		  bind(int fd, sockaddr * addr, socklen_t len)
		  
		  示例：
		  struct sockaddr_in name;
		  bind (sock, (struct sockaddr *) &name, sizeof (name))

		- listen

		  int listen (int socketfd, int backlog)

		- accept

		  int accept(int listensockfd, struct sockaddr *cliaddr, socklen_t *addrlen)

		- connect

		  int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen)

		- read

		  ssize_t read (int socketfd, void *buffer, size_t size)

		- write

		  ssize_t write (int socketfd, const void *buffer, size_t size)
		  
		  ssize_t send (int socketfd, const void *buffer, size_t size, int flags)
		  
		  ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags)

		- close
		- 代码示例

			- 客户端

			  #define MESSAGE_SIZE 102400
			  
			  void send_data(int sockfd) {
			    char *query;
			    query = malloc(MESSAGE_SIZE + 1);
			    for (int i = 0; i < MESSAGE_SIZE; i++) {
			      query[i] = 'a';
			    }
			    query[MESSAGE_SIZE] = '\0';
			  
			    const char *cp;
			    cp = query;
			    size_t remaining = strlen(query);
			    while (remaining) {
			      int n_written = send(sockfd, cp, remaining, 0);
			      fprintf(stdout, "send into buffer %ld \n", n_written);
			      if (n_written <= 0) {
			        error(1, errno, "send failed");
			        return;
			      }
			      remaining -= n_written;
			      cp += n_written;
			    }
			  
			    return;
			  }
			  
			  int main(int argc, char **argv) {
			    int sockfd;
			    struct sockaddr_in servaddr;
			  
			    if (argc != 2)
			      error(1, 0, "usage: tcpclient <IPaddress>");
			  
			    sockfd = socket(AF_INET, SOCK_STREAM, 0);
			  
			    bzero(&servaddr, sizeof(servaddr));
			    servaddr.sin_family = AF_INET;
			    servaddr.sin_port = htons(12345);
			    inet_pton(AF_INET, argv[1], &servaddr.sin_addr);
			    int connect_rt = connect(sockfd, (struct sockaddr *) &servaddr, sizeof(servaddr));
			    if (connect_rt < 0) {
			      error(1, errno, "connect failed ");
			    }
			    send_data(sockfd);
			    exit(0);
			  }

			- 服务器

			  /* 从socketfd描述字中读取"size"个字节. */
			  size_t readn(int fd, void *buffer, size_t size) {
			    char *buffer_pointer = buffer;
			    int length = size;
			  
			    while (length > 0) {
			      int result = read(fd, buffer_pointer, length);
			  
			      if (result < 0) {
			        if (errno == EINTR)
			          continue;   /* 考虑非阻塞的情况，这里需要再次调用read */
			        else
			          return (-1);
			      } else if (result == 0)
			        break;        /* EOF(End of File)表示套接字关闭 */
			  
			      length -= result;
			      buffer_pointer += result;
			    }
			    return (size - length);    /* 返回的是实际读取的字节数*/
			  }
			  
			  void read_data(int sockfd) {
			    ssize_t n;
			    char buf[1024];
			  
			    int time = 0;
			    for (;;) {
			      fprintf(stdout, "block in read\n");
			      if ((n = readn(sockfd, buf, 1024)) == 0)
			        return;
			  
			      time++;
			      fprintf(stdout, "1K read for %d \n", time);
			      usleep(1000);
			    }
			  }
			  
			  
			  int main(int argc, char **argv) {
			    int listenfd, connfd;
			    socklen_t clilen;
			    struct sockaddr_in cliaddr, servaddr;
			  
			    listenfd = socket(AF_INET, SOCK_STREAM, 0);
			  
			    bzero(&servaddr, sizeof(servaddr));
			    servaddr.sin_family = AF_INET;
			    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
			    servaddr.sin_port = htons(12345);
			  
			    /* bind到本地地址，端口为12345 */
			    bind(listenfd, (struct sockaddr *) &servaddr, sizeof(servaddr));
			    /* listen的backlog为1024 */
			    listen(listenfd, 1024);
			  
			    /* 循环处理用户请求 */
			    for (;;) {
			      clilen = sizeof(cliaddr);
			      connfd = accept(listenfd, (struct sockaddr *) &cliaddr, &clilen);
			      read_data(connfd);  /* 读取数据 */
			      close(connfd);     /* 关闭连接套接字，注意不是监听套接字*/
			    }
			  }

	- 结构

		- 通用套接字地址

		  /* POSIX.1g 规范规定了地址族为2字节的值. */
		  typedef unsigned short int sa_family_t;
		  /* 描述通用套接字地址 */
		  struct sockaddr{
		    sa_family_t sa_family; /* 地址族. 16-bit*/
		    char sa_data[14];  /* 具体的地址值 112-bit */
		   };

		- IPv4套接字地址

		  /* IPV4套接字地址，32bit值. */
		  typedef uint32_t in_addr_t;
		  struct in_addr
		   {
		    in_addr_t s_addr;
		   };
		    
		  /* 描述IPV4的套接字地址格式 */
		  struct sockaddr_in
		   {
		    sa_family_t sin_family; /* 16-bit */
		    in_port_t sin_port;   /* 端口口 16-bit*/
		    struct in_addr sin_addr;  /* Internet address. 32-bit */
		  
		  
		    /* 这里仅仅用作占位符，不做实际用处 */
		    unsigned char sin_zero[8];
		   };

		- IPv6套接字地址

		  struct sockaddr_in6
		   {
		    sa_family_t sin6_family; /* 16-bit */
		    in_port_t sin6_port; /* 传输端口号 # 16-bit */
		    uint32_t sin6_flowinfo; /* IPv6流控信息 32-bit*/
		    struct in6_addr sin6_addr; /* IPv6地址128-bit */
		    uint32_t sin6_scope_id; /* IPv6域ID 32-bit */
		   };

		- 本地套接字地址

		  struct sockaddr_un {
		    unsigned short sun_family; /* 固定为 AF_LOCAL */
		    char sun_path[108];  /* 路径名 */
		  };

		- 保留端口

		  /* Standard well-known ports. */
		  enum
		   {
		    IPPORT_ECHO = 7,  /* Echo service. */
		    IPPORT_DISCARD = 9,  /* Discard transmissions service. */
		    IPPORT_SYSTAT = 11,  /* System status service. */
		    IPPORT_DAYTIME = 13, /* Time of day service. */
		    IPPORT_NETSTAT = 15, /* Network status service. */
		    IPPORT_FTP = 21,  /* File Transfer Protocol. */
		    IPPORT_TELNET = 23,  /* Telnet protocol. */
		    IPPORT_SMTP = 25,  /* Simple Mail Transfer Protocol. */
		    IPPORT_TIMESERVER = 37, /* Timeserver service. */
		    IPPORT_NAMESERVER = 42, /* Domain Name Service. */
		    IPPORT_WHOIS = 43,  /* Internet Whois service. */
		    IPPORT_MTP = 57,
		  
		  
		  
		  
		    IPPORT_TFTP = 69,  /* Trivial File Transfer Protocol. */
		    IPPORT_RJE = 77,
		    IPPORT_FINGER = 79,  /* Finger service. */
		    IPPORT_TTYLINK = 87,
		    IPPORT_SUPDUP = 95,  /* SUPDUP protocol. */
		  
		  
		    IPPORT_EXECSERVER = 512, /* execd service. */
		    IPPORT_LOGINSERVER = 513, /* rlogind service. */
		    IPPORT_CMDSERVER = 514,
		    IPPORT_EFSSERVER = 520,
		  
		  
		    /* UDP ports. */
		    IPPORT_BIFFUDP = 512,
		    IPPORT_WHOSERVER = 513,
		    IPPORT_ROUTESERVER = 520,
		  
		  
		    /* Ports less than this value are reserved for privileged processes. */
		    IPPORT_RESERVED = 1024,
		  
		  
		    /* Ports greater this value are reserved for (non-privileged) servers. */
		    IPPORT_USERRESERVED = 5000

	- 技术

		- 三次握手

		  客户端的协议栈向服务器端发送了 SYN 包，并告诉服务器端当前发送序列号 j，客户端进入 SYNC_SENT 状态；
		  服务器端的协议栈收到这个包之后，和客户端进行 ACK 应答，应答的值为 j+1，表示对 SYN 包 j 的确认，同时服务器也发送一个 SYN 包，告诉客户端当前我的发送序列号为 k，服务器端进入 SYNC_RCVD 状态；
		  客户端协议栈收到 ACK 之后，使得应用程序从 connect 调用返回，表示客户端到服务器端的单向连接建立成功，客户端的状态为 ESTABLISHED，同时客户端协议栈也会对服务器端的 SYN 包进行应答，应答数据为 k+1；
		  应答包到达服务器端后，服务器端协议栈使得 accept 阻塞调用返回，这个时候服务器端到客户端的单向连接也建立成功，服务器端也进入 ESTABLISHED 状态。
		  
		  形象一点的比喻是这样的，有 A 和 B 想进行通话：
		  A 先对 B 说：“喂，你在么？我在的，我的口令是 j。”
		  B 收到之后大声回答：“我收到你的口令 j 并准备好了，你准备好了吗？我的口令是 k。”
		  A 收到之后也大声回答：“我收到你的口令 k 并准备好了，我们开始吧。”

- IP

	- 分类

		- 类型

			- IPv4
			- IPv6

		- 地址范围

			- A类地址

			  10.0.0.0 - 10.255.255.255

			- B类地址

			  172.16.0.0 - 172.31.255.255

			- C类地址

			  192.168.0.0 - 192.168.255.255

			- D类地址

			  224.0.0.0 - 239.255.255.255
			  广播地址

	- 子网掩码

	  网络地址位数由子网掩码决定。
	  
	  子网掩码的格式永远是二进制格式：前面一连串的1，后面跟着一连串的0。
	  
	  为了更加直观，将一个斜线放在IP地址后面，接着用一个十进制数字用于表示网络的位数。如192.0.2.12/30，表示子网掩码前30位为1，后2位为0，所以主机个数是2^2=4。

### UDP

又叫数据报套接字(Datagram Socket)

SOCK_DGRAM

无连接的数据报协议。速度块。

协议本身不够可靠。需要应用程序进行设计处理，比如对报文进行编号，设计Request-Ack机制，再加上重传等，在一定程度上可以达到更为高可靠的UDP程序。当然，这种可靠性和TCP相比还有一定距离，但也可以弥补实战中UDP的一些不足。

- 函数

	- socket
	- bind
	- sendto

	  #include <sys/socket.h>
	  
	  ssize_t sendto(int sockfd, const void *buff, size_t nbytes, int flags,
	          const struct sockaddr *to, socklen_t addrlen);

	- recvfrom

	  #include <sys/socket.h>
	  
	  ssize_t recvfrom(int sockfd, void *buff, size_t nbytes, int flags, 
	  struct sockaddr *from, socklen_t *addrlen);

	- 代码示例

		- 客户端

		  #include "lib/common.h"
		  
		  # define  MAXLINE   4096
		  
		  int main(int argc, char **argv) {
		    if (argc != 2) {
		      error(1, 0, "usage: udpclient <IPaddress>");
		    }
		     
		    int socket_fd;
		    socket_fd = socket(AF_INET, SOCK_DGRAM, 0);
		  
		    struct sockaddr_in server_addr;
		    bzero(&server_addr, sizeof(server_addr));
		    server_addr.sin_family = AF_INET;
		    server_addr.sin_port = htons(SERV_PORT);
		    inet_pton(AF_INET, argv[1], &server_addr.sin_addr);
		  
		    socklen_t server_len = sizeof(server_addr);
		  
		    struct sockaddr *reply_addr;
		    reply_addr = malloc(server_len);
		  
		    char send_line[MAXLINE], recv_line[MAXLINE + 1];
		    socklen_t len;
		    int n;
		  
		    while (fgets(send_line, MAXLINE, stdin) != NULL) {
		      int i = strlen(send_line);
		      if (send_line[i - 1] == '\n') {
		        send_line[i - 1] = 0;
		      }
		  
		      printf("now sending %s\n", send_line);
		      size_t rt = sendto(socket_fd, send_line, strlen(send_line), 0, (struct sockaddr *) &server_addr, server_len);
		      if (rt < 0) {
		        error(1, errno, "send failed ");
		      }
		      printf("send bytes: %zu \n", rt);
		  
		      len = 0;
		      n = recvfrom(socket_fd, recv_line, MAXLINE, 0, reply_addr, &len);
		      if (n < 0)
		        error(1, errno, "recvfrom failed");
		      recv_line[n] = 0;
		      fputs(recv_line, stdout);
		      fputs("\n", stdout);
		    }
		  
		    exit(0);
		  }

		- 服务器

		  #include "lib/common.h"
		  
		  static int count;
		  
		  static void recvfrom_int(int signo) {
		    printf("\nreceived %d datagrams\n", count);
		    exit(0);
		  }
		  
		  
		  int main(int argc, char **argv) {
		    int socket_fd;
		    socket_fd = socket(AF_INET, SOCK_DGRAM, 0);
		  
		    struct sockaddr_in server_addr;
		    bzero(&server_addr, sizeof(server_addr));
		    server_addr.sin_family = AF_INET;
		    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
		    server_addr.sin_port = htons(SERV_PORT);
		  
		    bind(socket_fd, (struct sockaddr *) &server_addr, sizeof(server_addr));
		  
		    socklen_t client_len;
		    char message[MAXLINE];
		    count = 0;
		  
		    signal(SIGINT, recvfrom_int);
		  
		    struct sockaddr_in client_addr;
		    client_len = sizeof(client_addr);
		    for (;;) {
		      int n = recvfrom(socket_fd, message, MAXLINE, 0, (struct sockaddr *) &client_addr, &client_len);
		      message[n] = 0;
		      printf("received %d bytes: %s\n", n, message);
		  
		      char send_line[MAXLINE];
		      sprintf(send_line, "Hi, %s", message);
		  
		      sendto(socket_fd, send_line, strlen(send_line), 0, (struct sockaddr *) &client_addr, client_len);
		  
		      count++;
		    }
		  
		  }

### HTTP

- 版本

	- 0.9
	- 1.0
	- 1.1
	- 2.0

- 方法

	- GET
	- POST
	- DELETE
	- PUT
	- PATCH

## 编程

### C

- 指针

### C++

- 引用
- 友元函数
- 虚函数
- 模版
- STL
- OOP

	- 对象
	- 多态
	- 继承
	- 设计模式

### Go

- 框架

	- Gin
	- Echo
	- Beego

- 多线程

	- 通道
	- 协程

## Git

### 类型

- tree
- commit
- blob

### 命令

- config
- add
- commit
- rebase
- checkout
- branch
- remote
- reset
- push
- pull
- fetch

## 系统

### Linux

- 技术

	- CGroup
	- NameSpace

- 分支

	- RedHat
	- CentOS
	- ……

### MacOS

### Windows

## Docker

### NameSpace

### CGroup

## Kubernetes

