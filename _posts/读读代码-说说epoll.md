---
title: 读读代码--说说epoll
date: 2017-11-28 23:16:31
tags:
- C语言
- 读读代码
- I/O多路复用
---

因为自己的项目经历不是很经典，所以，从现在开始，除了刷算法题、看书以外，又多了一件事情--**阅读别人的代码**。在导师的建议下，并且我也觉得阅读别人的代码可以快速的提高自己的水平，不仅可以复习理论知识，也可以锻炼实践的能力，进而最后提高项目的质量。所以这件事我打算坚持下去。

今天我阅读的是关于epoll的一个例子，[源代码点击这里](https://github.com/o0myself0o/epoll)。

<!-- more -->

好的，针对这位兄弟的代码，我给出代码详细的分析：

```c++
#include <sys/socket.h>
#include <sys/epoll.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>

#define MAXLINE		10
#define OPEN_MAX	100
#define LISTENQ		20
#define SERV_PORT	5555
#define INFTIM		1000

void setnonblocking(int sock)
{
	int opts;

	/*
    fcntl()函数
    作用：可以改变已打开的文件性质
    返回值：fcntl的返回值与第二个参数有关。但是，如果fcntl执行出错，所有的命令都返回-1。
    参数二：F_GETFL的作用是取得文件描述符sock的文件状态标志，例如：只读打开、只写打开、非阻塞模式等等。其他的文件            状态标志可以查看open()函数。
	*/
	opts = fcntl(sock, F_GETFL);

	if(opts < 0) {
		perror("fcntl(sock, GETFL)");
		exit(1);
	}

	opts = opts | O_NONBLOCK; // 给文件状态标志按位或一个非阻塞状态标志，从而让该文件变为非阻塞的。

	if(fcntl(sock, F_SETFL, opts) < 0) {
		perror("fcntl(sock, SETFL, opts)");
		exit(1);
	}
}

int main(int argc, char *argv[])
{
	printf("epoll socket begins.\n");
	int i, maxi, listenfd, connfd, sockfd, epfd, nfds;
	ssize_t n; // ssize_t是signed size_t
	char line[MAXLINE];
	socklen_t clilen; // 将保存 struct sockaddr_un 结构的长度的变量类型，由 int 类型改为 socklen_t 类型
    
    //ev用于注册事件, events数组用于回传要处理的事件
	struct epoll_event ev, events[20]; // 结构体epoll_event被用于注册所感兴趣的事件（例如在函数epoll_ctl中使用）和回传所发生待处理的事件（例如在epoll_wait中使用）

    /* 
    epoll_create()函数
    作用：生成一个epoll专用的文件描述符。
    它其实是在内核申请一空间，用来存放我们想关注的socket fd上是否发生以及发生了什么事件。也就是说，这个epfd和我们需要监听的socket fd 是有关联的。

    参数一：指定我们在这个epoll fd上能关注的最大socket fd数< 在epoll_ctl()函数中会指定需要监听的socket fd >。size大小随我们定，只要有空间。
    注意：在linux-2.4.32内核中根据size大小初始化哈希表的大小，在linux2.6.10内核中该参数无用，使用红黑树管理所有的需要监听的文件描述符，而不是hash。
    */
	epfd = epoll_create(256);

    /*
    程序员不应操作sockaddr，sockaddr是给操作系统用的
    程序员应使用sockaddr_in来表示地址，sockaddr_in区分了地址和端口，使用更方便。
    */
	struct sockaddr_in clientaddr; // sockaddr_in结构体用来处理网络通信的地址
	struct sockaddr_in serveraddr;
 
    /*
    AF_INET是 IPv4 网络协议的套接字类型，选择 AF_INET 的目的就是使用 IPv4 进行通信。
    */
	listenfd = socket(AF_INET, SOCK_STREAM, 0);

	setnonblocking(listenfd);

	ev.data.fd = listenfd; // 设置与要处理的事件相关的文件描述符
	/*
    EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）。
    EPOLLIN事件只有当对端有数据写入时才会触发，所以触发一次后需要不断读取所有数据直到读完EAGAIN为止，否则剩下的数据只有在下次对端有写入时才能一起取出来了。
    
    EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式。
	*/
	ev.events = EPOLLIN | EPOLLET; // 设置要处理的事件类型
    
    /*
    epoll_ctl()函数
    作用：用于控制某个文件描述符上的事件，可以注册事件，修改事件，删除事件。
    参数一：epoll_create()的返回值
    参数二：表示动作，用三个宏来表示：
        EPOLL_CTL_ADD：注册新的fd到epfd中；
        EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
        EPOLL_CTL_DEL：从epfd中删除一个fd；
    参数三：关联的文件描述符；
    参数四：告诉内核需要监听什么事件
    */
	epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev); // 注册epoll事件。

	bzero(&serveraddr, sizeof(serveraddr));
	serveraddr.sin_family = AF_INET;
	char *local_addr = "192.168.199.8";
	inet_aton(local_addr, &(serveraddr.sin_addr)); // // 将一个字符串IP地址转换为一个32位的网络序列IP地址
	serveraddr.sin_port = htons(SERV_PORT); // 将整型变量从主机字节顺序转变成网络字节顺序

	bind(listenfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));

	listen(listenfd, LISTENQ); // LISTENQ未经过处理的连接请求队列可以容纳的最大数目

	maxi = 0;

	for(; ;) {
		/*
		epoll_wait()函数
		作用：用于轮询I/O事件的发生，等待事件触发，当超过timeout还没有事件触发时，就超时。
		返回值：该函数返回需要处理的事件数目，如返回0表示已超时。
               返回的事件集合在events数组中，数组中实际存放的成员个数等于函数的返回值。返回0表示已经超时。

		参数一：由epoll_create 生成的epoll专用的文件描述符。
        参数二：events用来保存从内核中得到事件的集合(即，用于回传代处理事件的数组)。
        参数三：maxevents告之内核这个events有多大(数组成员的个数)，每次能处理的事件数。这个maxevents的值不能大于创建epoll_create()时的size。
        参数四：等待I/O事件发生的超时值

        epoll_wait运行的原理：
		等侍注册在epfd上的socket fd的事件的发生，如果发生则将发生的sokct fd和事件类型放入到events数组中。
		并 且将注册在epfd上的socket fd的事件类型给清空，所以如果下一个循环你还要关注这个socket fd的话，则需要用epoll_ctl(epfd,EPOLL_CTL_MOD,listenfd,&ev)来重新设置socket fd的事件类型。这时不用EPOLL_CTL_ADD,因为socket fd并未清空，只是事件类型清空。
		*/
		nfds = epoll_wait(epfd, events, 20, 500);

		for(i = 0; i < nfds; ++i) {
			if(events[i].data.fd == listenfd) { // 此时的listenfd返回回来了，说明它可读了。
				printf("accept connection, fd is %d\n", listenfd);
				connfd = accept(listenfd, (struct sockaddr *)&clientaddr, &clilen);
				if(connfd < 0) {
					perror("connfd < 0");
					exit(1);
				}

				setnonblocking(connfd);

				char *str = inet_ntoa(clientaddr.sin_addr);
				printf("connect from %s\n", str);

				ev.data.fd = connfd;
				ev.events = EPOLLIN | EPOLLET;
				epoll_ctl(epfd, EPOLL_CTL_ADD, connfd, &ev);
			}
			else if(events[i].events & EPOLLIN) { // 如果返回的这个events[i]可读并且不是listenfd
				if((sockfd = events[i].data.fd) < 0) continue;
				if((n = read(sockfd, line, MAXLINE)) < 0) { // 开始从sockfd中读取发送过来的数据
					if(errno == ECONNRESET) {
						close(sockfd);
						events[i].data.fd = -1;
					} else {
						printf("readline error");
					}
				} else if(n == 0) {
					close(sockfd);
					events[i].data.fd = -1;
				}
				
				printf("received data: %s\n", line);

				ev.data.fd = sockfd;

				/*
                ET模式下，EPOLLOUT触发条件有：
				1.缓冲区满-->缓冲区非满；例如：对端读取了一些数据，又重新可写了，此时会触发EPOLLOUT。
				2.同时监听EPOLLOUT和EPOLLIN事件 时，当有IN 事件发生，都会顺带一个OUT事件；
				3.一个客户端connect过来，accept成功后会触发一次OUT事件。
				*/
				ev.events = EPOLLOUT | EPOLLET; // EPOLLOUT表示对应的文件描述符可写
				epoll_ctl(epfd, EPOLL_CTL_MOD, sockfd, &ev);
			}
			else if(events[i].events & EPOLLOUT) { // 如果返回的这个events[i]可写并且不是listenfd
				sockfd = events[i].data.fd;
				write(sockfd, line, n);

				printf("written data: %s\n", line);

				ev.data.fd = sockfd;
				ev.events = EPOLLIN | EPOLLET;
				epoll_ctl(epfd, EPOLL_CTL_MOD, sockfd, &ev);
			}
		}
	}
}
```

对于上面的代码，有一些地方必须进行特别的说明，我们从上往下来说。（这些是我在阅读的时候对自己所提出的问题）

一、代码中有两处地方调用了`setnonblocking`函数。这个函数的作用是设置文件描述符为非阻塞状态。那么为什么需要设置它为非阻塞的呢？其实是有原因的。

我们会发现，在代码中的一些地方对文件描述符设置了要处理的事件类型，它们都被设置为了EPOLLET（边沿触发模式）。我们可以把边沿模式理解为：**不到边缘情况，是死都不会触发的**。

原因可以参考下面的链接，就不复制粘贴里面的内容了，复制粘贴很没意思：

[彻底学会使用epoll(六)——关于ET的若干问题总结](http://blog.csdn.net/weiyuefei/article/details/52242890)

[实例浅析epoll的水平触发和边缘触发，以及边缘触发为什么要使用非阻塞IO](http://www.cnblogs.com/yuuyuu/p/5103744.html)

[使用epoll时需要将socket设为非阻塞吗？](https://www.zhihu.com/question/23614342)

二、在调用函数`epoll_ctl`，参数3已经指定了要监听的fd，那么为什么ev.data.fd还需要指明，是否多余？

对于epoll，还需要更多的理解，需要寻找更多的源码来运行，看看结果，然后分析。

happy ending......

三、

```c++
bzero(&serveraddr, sizeof(serveraddr));
```

这是**一个细节**，用来初始化服务器端（自己）的地址和端口信息。