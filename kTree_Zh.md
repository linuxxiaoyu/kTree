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

### 套接字

socket

- 类型

	- 通用套接字地址

	  ```c
	  /* POSIX.1g 规范规定了地址族为2字节的值. */
	  typedef unsigned short int sa_family_t;
	  /* 描述通用套接字地址 */
	  struct sockaddr{
	    sa_family_t sa_family; /* 地址族. 16-bit*/
	    char sa_data[14];  /* 具体的地址值 112-bit */
	   }; 
	  ```

	- IPv4套接字地址

	  ```c
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
	  ```

	- IPv6套接字地址

	  ```c
	  struct sockaddr_in6
	   {
	    sa_family_t sin6_family; /* 16-bit */
	    in_port_t sin6_port; /* 传输端口号 # 16-bit */
	    uint32_t sin6_flowinfo; /* IPv6流控信息 32-bit*/
	    struct in6_addr sin6_addr; /* IPv6地址128-bit */
	    uint32_t sin6_scope_id; /* IPv6域ID 32-bit */
	   };
	  ```

	- 本地套接字地址

	  ```c
	  struct sockaddr_un {
	    unsigned short sun_family; /* 固定为 AF_LOCAL */
	    char sun_path[108];  /* 路径名 */
	  };
	  ```

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

		  ```c
		  int socket(int domain, int type, int protocol)
		  ```
		  
		  domain：
		  PF_INET：IPv4
		  PF_INET6：IPv6
		  PF_LOCAL：本地
		  
		  type：
		  SOCK_STREAM：表示字节流，对应TCP
		  SOCK_DGRAM：表示数据报，对应UDP
		  SOCK_RAW：表示原始套接字

		- bind

		  ```c
		  bind(int fd, sockaddr * addr, socklen_t len)
		  ```
		  
		  示例：
		  struct sockaddr_in name;
		  bind (sock, (struct sockaddr *) &name, sizeof (name))

		- listen

		  ```c
		  int listen (int socketfd, int backlog)
		  ```

		- accept

		  ```c
		  int accept(int listensockfd, struct sockaddr *cliaddr, socklen_t *addrlen)
		  ```

		- connect

		  ```c
		  int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen)
		  ```

		- read

		  ```c
		  ssize_t read (int socketfd, void *buffer, size_t size)
		  ```

		- write

		  ```c
		  ssize_t write (int socketfd, const void *buffer, size_t size);
		  
		  ssize_t send (int socketfd, const void *buffer, size_t size, int flags);
		  
		  ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
		  ```

		- close
		- 代码示例

			- 客户端

			  ```c
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
			  ```

			- 服务器

			  ```c
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
			  ```

	- 保留端口

	  ```c
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
	  ```

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

		- 四次挥手

		  TCP连接终止时，主机1先发送FIN报文，主机2进入CLOSE_WAIT状态，先发送一个ACK应答，同时，主机2通过read调用获取EOF，并将此结果通知应用程序进行主动关闭操作，发送FIN报文。主机1在接收到FIN报文后发送ACK应答，此时主机1进入TIME_WAIT状态。
		  
		  主机1在TIME_WAIT停留持续时间是固定的，是最长分节生命期MSL（maximum segment lifetime）的两倍，一般称为2MSL。Linux系统里有一个硬编码的字段，名称为TCP_TIMEWAIT_LEN，其值为60秒。也就是说，Linux系统停留在TIME_WAIT的时间为固定的60秒。

		- TIME_WAIT

		  只有发起连接终止的一方会进入TIME_WAIT状态。

			- 作用

				- 确保最后的ACK能让被动关闭方接收，从而帮助其正常关闭。
				- 和连接“化身”、报文迷走有关，为了让旧连接的重复分节在网络中自然消失。

			- 危害

				- 占用内存资源
				- 占用端口资源。如果TIME_WAIT状态过多，会导致无法创建新连接。

			- 优化

				- net.ipv4.tcp_tw_reuse（推荐做法）

					- 从协议角度理解如果是安全可控的，可以复用处于TIME_WAIT的套接字为新的连接所用。需要打开对TCP时间戳的支持，即net.ipv4.tcp_timestamps=1（默认即为 1）。

				- net.ipv4.tcp_max_tw_buckets

					- 一个暴力的方法是通过 sysctl 命令，将系统值调小。这个值默认为 18000，当系统中处于 TIME_WAIT 的连接一旦超过这个值时，系统就会将所有的 TIME_WAIT 连接状态重置，并且只打印出警告信息。这个方法过于暴力，而且治标不治本，带来的问题远比解决的问题多，不推荐使用。

				- 调低 TCP_TIMEWAIT_LEN，重新编译系统

					- 需要内核知识，要重新编译内核。不具有普遍性

				- SO_LINGER 的设置

					- 是一个非常危险的行为，不值得提倡。

					  如果l_onoff为非 0， 且l_linger值也为 0，那么调用 close 后，会立该发送一个 RST 标志给对端，该 TCP 连接将跳过四次挥手，也就跳过了 TIME_WAIT 状态，直接关闭。这种关闭的方式称为“强行关闭”。 在这种情况下，排队数据不会被发送，被动关闭方也不知道对端已经彻底断开。只有当被动关闭方正阻塞在recv()调用上时，接受到 RST 时，会立刻得到一个“connet reset by peer”的异常。
					  
					  ```c
					  struct linger so_linger;
					  so_linger.l_onoff = 1;
					  so_linger.l_linger = 0;
					  setsockopt(s,SOL_SOCKET,SO_LINGER, &so_linger,sizeof(so_linger));
					  ```

		- 关闭连接

			- close

			  close 函数只是把套接字引用计数减 1，未必会立即关闭连接；
			  close 函数如果在套接字引用计数达到 0 时，立即终止读和写两个方向的数据传送。

			- shutdown

			  在期望关闭连接其中一个方向时，应该使用 shutdown 函数。

			- 对比

			  客户端调用close函数关闭了整个连接，当服务器端发送“Hi, data1”时，客户端给服务器端回送了一个RST分组；服务器端再次尝试发送"Hi, data2"时，系统内核通知SIGPIPE信号。这是因为，在RST的套接字进行写操作，会直接出发SIGPIPE信号。需要对SIGPIPE信号进行处理，来避免程序莫名退出：
			  ```c
			  static void sig_pipe(int signo) {
			    printf("\nreceived %d datagrams\n", count);
			    exit(0);
			  }
			  signal(SIGINT, sig_pipe);
			  ```

		- Keep-Alive

		  定义一个时间段，在这个时间段内，如果没有任何连接相关的活动，TCP保活机制会开始作用，每隔一个时间间隔，发送一个探测报文，该探测报文包含的数据非常少，如果连续几个探测报文都没有相应，则认为当前的TCP连接已经死亡，系统内核将错误信息通知给上层应用程序。
		  
		  上述的可定义变量，分别被称为保活时间、保活时间间隔和保活探测次数。在Linux系统中，这些变量分别对应sysctl变量net.ipv4.tcp_keepalive_time、net.ipv4.tcp_keepalive_intvl、net.ipv4.tcp_keepalive_probes，默认设置是7200秒，75秒和9次探测。
		  
		  如果开启了TCP保活，需要考虑以下情况：
		  对端程序正常工作。当TCP保活的探测报文发送给对端，对端会正常响应，这样TCP保活时间会被重置，等待下一个TCP保活时间的到来。
		  对端程序崩溃并重启。当TCP保活的探测报文发送给对端后，对端是可以响应的，但由于没有该连接的有效信息，会产生一个RST报文，这样很快就会发现TCP连接已经被重置。
		  对端程序崩溃，或者对端由于其他原因导致报文不可达。当TCP保活的探测报文发送给对端后，石沉大海，没有响应，连续几次，达到保活探测次数后，TCP会报告该TCP连接已经死亡。
		  
		  TCP保活机制默认是关闭的。在Linux系统中，最少需要经过7200+75*9=2小时11分15秒才可以发现一个“死亡”连接。实际上，对很多对时延要求敏感的系统来说，这个时间间隔是不可接受的。
		  
		  我们可以通过在应用程序中模拟TCP Keep-Alive机制，来完成应用层的连接探活，比如设计一个PING-PONG的机制。

		- 接收窗口
		- 发送窗口
		- 拥塞窗口

		  拥塞控制：多个连接共享在有限的带宽上时，为了兼顾效率和公平性的控制。
		  
		  拥塞控制通过拥塞窗口来完成。
		  拥塞窗口的大小会随着网络状况实时调整。

			- 慢启动

			  通过一定规则，慢慢的将网络发送数据的速率增加到一个阀值。超过这个阀值之后，慢启动就结束了。

			- 拥塞避免

			  慢启动结束后，会进行拥塞避免。
			  TCP会不断的探测网络状况，并随之不断调整拥塞窗口的大小。

		- Nagle算法

		  限制大批量的小数据包同时发送。
		  它提出在任何时刻，未被确认的小数据包不能超过一个。
		  小数据包指的是长度小于最大报文长度MSS的TCP分组。
		  发送端可以把接下来连续的几个小数据包存储起来，等待接收到前一个小数据包的ACK分组后，再将数据一次性发送出去。

		- 延时ACK机制

		  收到数据后并不马上回复，而是累计需要发送的ACK报文，等到有数据需要发送给对端时，将累计的ACK捎带一并发送出去。
		  延时ACK机制不能无限的延时下去，否则发送端误认为数据包没有发送成功，引起重传，反而会占用额外的网络带宽。

		- Tips

		  调用发送函数并不意味着数据会被真正发送到网络上，其实这些数据只是从应用程序中被拷贝到了系统内核的套接字缓冲区中，或者说是发送缓冲区中，等待协议栈的处理。至于这些数据是什么时候被发送出去的，对应用程序来说是无法预知的。对于这件事情真正负责的是，运行于操作系统内核的TCP协议栈实现模块。
		  任何时刻，TCP发送缓冲区的数据是否真正发送出去，至少取决于两个因素，一个是当前的发送窗口大小，另一个是拥塞窗口大小。TCP协议中总是取两者中最小值作为判断依据。
		  不要多次频繁的发送小报文，如果有，可以使用writev和readv函数进行批量发送和接收。
		  因为TIME_WAIT的存在，重启应用后可能得到“Address already in use”的错误信息。最佳实践为：服务器端程序，都应该设置SO_REUSEADDR套接字选项，以便服务端程序可以在极短时间内复用同一个端口启动。在所有TCP服务器程序中，调用bind之前请设置SO_REUSEADDR套接字选项。
		  
		  tcp_tw_reuse和SO_REUSEADDR的区别：
		  tcp_tw_reuse是内核选项，主要用在连接的发起方。TIME_WAIT状态的连接创建时间超过1秒后，新的连接才可以被复用。注意，这里是连接的发起方
		  SO_REUSEADDR是用户态的选项，SO_REUSEADDR选项用来告诉操作系统内核，如果端口已被占用，但是TCP连接状态位于TIME_WAIT，可以重用端口。如果端口忙，而TCP处于其他状态，重用端口时依旧得到“Address already in use”的错误信息。注意，这里一般都是连接的服务方。
		  
		   TCP并不是那么“可靠”的。可能会出现以下故障：
		  			1.对端无FIN包，需要通过巡检或超时来发现。
		  			2.对端有FIN包，需要通过增强read或write操作的异常处理，帮我们发现此类异常。
		  
		  通过对各种异常边界的检测，可以提高程序在恶劣环境下的稳定性。所以要时刻提醒自己做好应对各种复杂情况的准备，包括缓冲区溢出、指针错误、连接超时检测等。

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

	  ```c
	  #include <sys/socket.h>
	  
	  ssize_t sendto(int sockfd, const void *buff, size_t nbytes, int flags,
	          const struct sockaddr *to, socklen_t addrlen); 
	  ```

	- recvfrom

	  ```c
	  #include <sys/socket.h>
	  
	  ssize_t recvfrom(int sockfd, void *buff, size_t nbytes, int flags, 
	  struct sockaddr *from, socklen_t *addrlen); 
	  ```

	- 代码示例

		- 客户端

		  ```c
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
		  ```

		- 服务器

		  ```c
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
		  ```

- UDP连接

	- 实现方式

		- 对UDP调用connect，绑定本地地址和端口
		- 使用send或write函数来发送，如果使用sendto需要把相关的to地址信息置零
		- 使用recv或read函数来接收，如果使用recvfrom需要把对应的from地址信息置零

	- 作用

		- 快速获取异步错误信息的通知

			- 不使用connect创建UDP连接时，服务器不可用时，客户端会阻塞住。若使用connect创建了UDP连接，内核能够根据IP四元组找到对应的程序，并在读取或写入数据时报告错误。

		- 获得一定性能上的提升

			- 因为省去了每次都要连接和断开套接字的过程

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

### 分析工具

- ping

  探测网络连通性。基于ICMP协议开发。
  
  按照sequence序列号排序显示，一并显示的，也包括TTL(time to live)，反应了两个IP地址之间传输的时间。最后还显示了ping命令的统计信息，如最小时间，平均时间等。

- ifconfig

  显示当前系统中的所有网络设备。

- netstat

  查看活动的连接状态。
  
  查看当前所有连接状态：
  ```shell
  netstat -alepn
  ```

- lsof

  查看活动的连接状态。
  
  查看打开sock文件的进程：
  ```shell
  lsof /path/of/sock/1.sock
  ```
  
  查看占用端口的进程：
  ```shell
  lsof -i :8080
  ```

- tcpdump

  可以对各种奇怪环境进行抓包，进而了解报文，排查问题。
  
  指定网卡：
  ```shell
  tcpdump -i eth0
  ```
  
  指定来源：
  ```shell
  tcpdump src host hostname
  ```
  
  TCP，端口80，来自IP为192.168.1.25的主机：
  ```shell
  tcpdump 'tcp and port 80 and src host 192.168.1.25'
  ```
  
  输出格式：
  timestamp IP src.port > dst.port: Flags [……], …… 
  Flags[]是包的标志。常用格式如下：
  [S]: SYN, 表示开始连接
  [.]: 没有标记，一般是确认
  [P]: PSH，表示数据推送
  [F]: FIN，表示结束连接
  [R]: RST，表示重启连接

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

