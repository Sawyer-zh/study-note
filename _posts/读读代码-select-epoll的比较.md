---
title: 读读代码--select/epoll的比较
date: 2017-11-30 21:08:28
tags:
- C语言
- 读读代码
- I/O多路复用
---

今天，我阅读的代码是关于select和epoll这两种I/O多路复用方式的比较。通过代码对比和现象，我们可以更加深刻的理解这两种方式的区别。关于`select`我之前在 [I/O多路复用之select()系统调用](https://huanghantao.github.io/2017/09/13/I-O%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E4%B9%8Bselect-%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8/)中有说，这里就不重复了，而epoll在上一篇文章中也有说过，所以我们这里就简单讲一下I/O多路复用的一个比较明显的共同特点吧：都是通过I/O多路复用的API来**监听我们想要监听的文件描述符的某些事件发生，进而我们所操作的文件描述符都是已经就绪好了的，这样我们在使用accept正式操作的时候，阻塞发生的情况就更少了**。

OK，我们来读读代码。这次的代码是来自[这位哥们](https://github.com/rudy-yuan/select-epoll)。

首先，我们阅读的第一份代码是一个`select`客户端，它监听了两个文件描述符，一个是标准输入（即键盘），另一个是表示连接的socket文件描述符（用来和服务器通信的）。

<!-- more -->

```c++
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/socket.h>
#include <resolv.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>

#define MAXBUF 1024
/*********************************************************************
* filename: select-client.c
* 演示网络异步通讯，这是客户端程序
*********************************************************************/

int main(int argc, char **argv)
{
    int sockfd, len;
    
    struct sockaddr_in dest;
    char buffer[MAXBUF + 1];
    
    /*
    select()机制中提供一fd_set的数据结构，
    实际上是一long类型的数组，
    每一个数组元素都能与一打开的文件句柄（不管是socket句柄，还是其他文件或命名管道或设备句柄）建立联系，
    建立联系的工作由程序员完成，当调用select()时，
    由内核根据IO状态修改fd_set的内容，
    由此来通知执行了select()的进程哪一socket或文件发生了可读或可写事件。
    */
    fd_set rfds;
    struct timeval tv;
    
    int retval, maxfd = -1;

    if (argc != 3)
    {
        printf("参数格式错误！正确用法如下：\n\t\t%s IP地址 端口\n\t比如:\t%s 127.0.0.1 80\n此程序用来从某个 IP 地址的服务器某个端口接收最多 MAXBUF 个字节的消息", argv[0], argv[0]);
        exit(0);
    }
    
    // 创建一个 socket 用于 tcp 通信 
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) 
    {
        perror("Socket");
        exit(errno);
    }

    // 初始化服务器端（对方）的地址和端口信息
    bzero(&dest, sizeof(dest)); // 一个细节，需要注意
    dest.sin_family = AF_INET;
    dest.sin_port = htons(atoi(argv[2]));
    
    if (inet_aton(argv[1], (struct in_addr *) &dest.sin_addr.s_addr) == 0) 
    {
        perror(argv[1]);
        exit(errno);
    }

    // 连接服务器 
    if (connect(sockfd, (struct sockaddr *) &dest, sizeof(dest)) != 0) 
    {
        perror("Connect ");
        exit(errno);
    }

    printf("\n准备就绪，可以开始聊天了……直接输入消息回车即可发信息给对方\n");
    
    while (1) 
    {
        /*
        将指定的文件描述符集清空，在对文件描述符集合进行设置前，必须对其进行初始化。
        如果不清空，由于在系统分配内存空间后，通常并不作清空处理，所以结果是不可知的
        */
        FD_ZERO(&rfds); // 把集合清空
        
        FD_SET(0, &rfds); // 把标准输入句柄0加入到集合中
	
        maxfd = 0;
        
        FD_SET(sockfd, &rfds); // 把当前连接句柄sockfd加入到集合中
	
        if (sockfd > maxfd)
            maxfd = sockfd; // 这一步很重要，因为select()函数需要用到最大的那个文件描述符值
	
        // 设置最大等待时间 
        tv.tv_sec = 3;
        tv.tv_usec = 0;
	
        /*
        作用：当一个套接字或一组套接字有信号时通知你（即，告诉你有的文件描述符准备好了，但是并没有告诉你是哪一个文件描述符）。
        参数1：需要监视的最大的文件描述符值+1。
        参数2：需要检测的可读文件描述符的集合。
               readfds参数标识等待可读性检查的套接口。
               如果该套接口正处于监听listen()状态，则若有连接请求到达，该套接口便被标识为可读，
               这样一个accept()调用保证可以无阻塞完成

        参数3：需要检测的可写文件描述符的集合。
               writefds参数标识等待可写性检查的套接口。
               如果一个套接口正在connect()连接（非阻塞），可写性意味着连接顺利建立。
               如果套接口并未处于connect()调用中，可写性意味着send()和sendto()调用将无阻塞完成。〔但并未指出这个保证在多长时间内有效，特别是在多线程环境中〕。

        参数4：需要检测的异常文件描述符的集合。
        参数5：如果在这个时间内，需要监视的描述符没有事件发生则函数返回。
        */
        retval = select(maxfd + 1, &rfds, NULL, NULL, &tv); // 开始等待
	
        if (retval == -1) {
            printf("将退出，select出错！ %s", strerror(errno));
            break;
        } 
        else if (retval == 0) {
            /* printf("没有任何消息到来，用户也没有按键，继续等待……\n"); */
            continue;
        } 
        else {
            if (FD_ISSET(sockfd, &rfds)) { // 连接的socket上有消息到来则接收对方发过来的消息并显示（因为我们把表示连接的socket放在了需要检测的可读文件描述符的集合中，所以当成功返回的时候，说明该连接上面有消息了）
                bzero(buffer, MAXBUF + 1); // 注意：一个细节
                // 接收对方发过来的消息，最多接收 MAXBUF 个字节 
                len = recv(sockfd, buffer, MAXBUF, 0);
		
                if (len > 0) {
                    printf("接收消息成功:'%s'，共%d个字节的数据\n", buffer, len);
		        }
                else {
                    if (len < 0) {
                        printf("消息接收失败！错误代码是%d，错误信息是'%s'\n", errno, strerror(errno));
		            }
                    else {
                        printf("对方退出了，聊天终止！\n"); // 说明了一个问题：服务器退出的时候，该socket也是可读的
		            }

                    break;
                }
            }
            
            if (FD_ISSET(0, &rfds)) { // 用户按了键盘且回车了，则读取用户输入的内容发送出去
                bzero(buffer, MAXBUF + 1); // 注意：一个细节
                fgets(buffer, MAXBUF, stdin); // 读取一行。注意，stdin中包含了用户按下的回车(\n)，但是fgets不会把\n读进buffer中。且每次最多读取bufsize-1个字符（第bufsize个字符赋'\0'）
                if (!strncasecmp(buffer, "quit", 4)) {
                    printf("自己请求终止聊天！\n");
                    break;
                }
                
                // 发消息给服务器
                len = send(sockfd, buffer, strlen(buffer) - 1, 0); // 没有把buffer中的字符串结束符\0发送过去
                if (len < 0) {
                    printf("消息'%s'发送失败！错误代码是%d，错误信息是'%s'\n", buffer, errno, strerror(errno));
                    break;
                }
                else {
                    printf("消息：%s\t发送成功，共发送了%d个字节！\n", buffer, len);
		        }
            }
        }
    }
    
    // 关闭连接 
    close(sockfd);
    return 0;
}
```

（因为这份代码和上一篇文章的代码有一些地方是类似的，所以就省略了一些注释。）

这份代码整体来说不难理解，基本不会卡壳的。OK，我来说一说这份代码中的一些值得学习的地方。

一、初始化

代码中有两个地方有明显的初始化：

```c++
// 第一个地方
// 初始化服务器端（对方）的地址和端口信息
bzero(&dest, sizeof(dest)); // 一个细节，需要注意

// 第二个地方
FD_ZERO(&rfds); // 把集合清空

// 第三个地方
bzero(buffer, MAXBUF + 1); // 注意：一个细节
```

二、返回值的判断

对于那些需要进行返回值的判断的代码，作者都把函数调用直接放在了if判断语句里面。这样做可以**减少很多几乎不会再用到第二次的变量**。可以增强代码的可读性。

OK，今晚先看到这里，明天继续看。今晚开始看慕课网上的**《PHP高并发秒杀系统》**的视频咯。发现想学的东西太多太多......

导师建议以后学学人工智能的知识，赞同。但是我还是得先把计算机的基础打好，然后工作的闲暇时间再来学习，希望我的数学基础那时候还保留着......

-------------------------------------------------------------------------------------------------------------------------------------------

今天阅读了服务器的代码，是select和epoll两种模型的对比，我们先来看看支持select服务器的代码：

```c++
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/time.h>
#include <sys/types.h>

#define MAXBUF 1024

/*********************************************************************
* filename: select-server.c
* 演示网络异步通讯、select用法，这是服务器端程序
* ./server 7838 1
*********************************************************************/

int main(int argc, char **argv)
{
    int sockfd, new_fd;
    socklen_t len;
    struct sockaddr_in my_addr, their_addr;
    
    unsigned int myport, lisnum;
    char buf[MAXBUF + 1];
    
    fd_set rfds;
    struct timeval tv;
    
    int retval, maxfd = -1;

    if (argv[1])
        myport = atoi(argv[1]);
    else
        myport = 7838; // 如果没有指定服务器监听的端口，那么就使用这个默认的端口

    if (argv[2])
        lisnum = atoi(argv[2]);
    else
        lisnum = 2;

    // 创建一个socket文件描述符，作为捆绑使用
    if ((sockfd = socket(PF_INET, SOCK_STREAM, 0)) == -1) 
    {
        perror("socket");
        exit(1);
    }

    bzero(&my_addr, sizeof(my_addr));
    my_addr.sin_family = PF_INET;
    my_addr.sin_port = htons(myport);
    
    if (argv[3])
        my_addr.sin_addr.s_addr = inet_addr(argv[3]);
    else
        my_addr.sin_addr.s_addr = INADDR_ANY; // INADDR_ANY 指的是0.0.0.0

    if (bind(sockfd, (struct sockaddr *) &my_addr, sizeof(struct sockaddr)) == -1) 
    {
        perror("bind");
        exit(1);
    }

    if (listen(sockfd, lisnum) == -1) // sockfd 是一个已捆绑未连接套接口的描述字，lisnum是等待连接队列的最大长度。
    {
        perror("listen");
        exit(1);
    }

    while (1) { // 第一层循环可以用来从请求队列里面获取新的socket文件描述符
        printf("\n----等待新的连接到来开始新一轮聊天……\n");
	
        len = sizeof(struct sockaddr);
	
        if ((new_fd = accept(sockfd, (struct sockaddr *) &their_addr, &len)) == -1) {
            perror("accept");
            exit(errno);
        } 
        else {
            printf("server: got connection from %s, port %d, socket %d\n", inet_ntoa(their_addr.sin_addr), ntohs(their_addr.sin_port), new_fd);
	    }
	
        // 开始处理每个新连接上的数据收发
        printf("\n准备就绪，可以开始聊天了……直接输入消息回车即可发信息给对方\n");
	
        while (1) { // 第二次循环用来把那个新的socket文件描述符加入需要被监听的文件描述符集合中。（注意其他的socket文件描述符被清空了）
            // 把集合清空 
            FD_ZERO(&rfds); // 注意
	    
            // 把标准输入(stdin)句柄0加入到集合中
            FD_SET(0, &rfds);
    
            // 把当前连接(socket)句柄new_fd加入到集合中 
            FD_SET(new_fd, &rfds);
	    
	        maxfd = 0;
            if (new_fd > maxfd) {
                maxfd = new_fd;
	        }
	    
            // 设置最大等待时间 
            tv.tv_sec = 5;
            tv.tv_usec = 0;
	    
            // 开始等待 
            retval = select(maxfd + 1, &rfds, NULL, NULL, &tv);
	    
            if (retval == -1) {
                printf("将退出，select出错！ %s", strerror(errno));
                break;
            } 
            else if (retval == 0) { // 当select设置的时间到了，并且此时的new_fd并不可读，则retval == 0
                printf("没有任何消息到来，用户也没有按键，继续等待……\n");
                continue;
            } 
            else { // 当new_fd可读的时候，retval > 0。此时可能是stdin文件描述符可读，也可能是new_fd可读，也可能是都可读。
		        // 判断当前IO是否是stdin
                if (FD_ISSET(0, &rfds)) { // 用户按键了，则读取用户输入的内容发送出去                  
                    bzero(buf, MAXBUF + 1);
                    fgets(buf, MAXBUF, stdin);
                    if (!strncasecmp(buf, "quit", 4)) {
                        printf("自己请求终止聊天！\n");
                        break;
                    }
                    len = send(new_fd, buf, strlen(buf) - 1, 0);
                    if (len > 0)
                        printf("消息:%s\t发送成功，共发送了%d个字节！\n", buf, len);
                    else {
                        printf("消息'%s'发送失败！错误代码是%d，错误信息是'%s'\n", buf, errno, strerror(errno));
                        break;
                    }
                }
                
		        // 判断当前IO是否是来自socket
                if (FD_ISSET(new_fd, &rfds)) {  // 当前连接的socket上有消息到来则接收对方发过来的消息并显示                
                    bzero(buf, MAXBUF + 1);
                    // 接收客户端的消息 
                    len = recv(new_fd, buf, MAXBUF, 0);
                    if (len > 0) {
                        printf("接收消息成功:'%s'，共%d个字节的数据\n", buf, len);
		            }
                    else {
                        if (len < 0)
                            printf("消息接收失败！错误代码是%d，错误信息是'%s'\n", errno, strerror(errno));
                        else
                            printf("对方退出了，聊天终止\n");
                        break;
                    }
                }
            }
        }
        
        close(new_fd);
        // 处理每个新连接上的数据收发结束
	
        printf("还要和其它连接聊天吗？(no->退出)");
        fflush(stdout);
	
        bzero(buf, MAXBUF + 1);
        fgets(buf, MAXBUF, stdin);
        if (!strncasecmp(buf, "no", 2)) {
            printf("终止聊天！\n");
            break;
        }
    }

    close(sockfd);
    return 0;
}
```

接着是支持epoll服务器的代码：

```c++
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <sys/socket.h>
#include <fcntl.h>
#include <sys/epoll.h>

#define MAXBUF 1024
#define MAXEPOLLSIZE 10000

/*
 * 设置句柄为非阻塞方式
 */
int setnonblocking(int sockfd)
{
    if (fcntl(sockfd, F_SETFL, fcntl(sockfd, F_GETFD, 0)|O_NONBLOCK) == -1) 
    {
        return -1;
    }
    return 0;
}

/*********************************************************************
* filename: epoll-server.c
* 演示epoll接受海量socket并进行处理响应的方法
*********************************************************************/

int main(int argc, char **argv)
{
    int listenfd, connfd, epfd, sockfd, nfds, n, curfds;
    socklen_t len;
    
    struct sockaddr_in my_addr, their_addr;
    unsigned int myport, lisnum;
    char buf[MAXBUF + 1];
    
    // 声明epoll_event结构体的变量，ev用于注册事件，events数组用于回传要处理的事件
    struct epoll_event ev;
    struct epoll_event events[MAXEPOLLSIZE];

    if (argv[1])
        myport = atoi(argv[1]);
    else
        myport = 7838;

    if (argv[2])
        lisnum = atoi(argv[2]);
    else
        lisnum = 2;

    // 开启 socket 监听
    if ((listenfd = socket(PF_INET, SOCK_STREAM, 0)) == -1) 
    {
        perror("socket");
        exit(1);
    } else
        printf("socket 创建成功！\n");

    // 把socket设置为非阻塞方式
    setnonblocking(listenfd);

    bzero(&my_addr, sizeof(my_addr));
    my_addr.sin_family = PF_INET;
    my_addr.sin_port = htons(myport);

    if (argv[3])
        my_addr.sin_addr.s_addr = inet_addr(argv[3]);
    else
        my_addr.sin_addr.s_addr = INADDR_ANY;

    if (bind(listenfd, (struct sockaddr *) &my_addr, sizeof(struct sockaddr)) == -1) 
    {
        perror("bind");
        exit(1);
    } else
        printf("IP 地址和端口绑定成功\n");

    if (listen(listenfd, lisnum) == -1) 
    {
        perror("listen");
        exit(1);
    } else
        printf("开启服务成功！\n");
    
    // 上面的代码是常规的网络编程需要做的事情，现在开始就是和I/O多路复用有关。
    // 创建 epoll句柄，把监听 socket 加入到 epoll 集合里 */
    epfd = epoll_create(MAXEPOLLSIZE); /*epoll专用的文件描述符*/
    len = sizeof(struct sockaddr_in);
    ev.events = EPOLLIN|EPOLLET;
    ev.data.fd = listenfd;
    
    // 将listenfd注册到epoll事件
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev) < 0) {
        fprintf(stderr, "epoll set insertion error: fd=%d\n", listenfd);
        return -1;
    } else
        printf("监听 socket 加入 epoll 成功！\n");

    curfds = 1; // 需要监听的文件描述符的个数
    
    while (1) {
        // 第一层循环用来等待事件发生
        nfds = epoll_wait(epfd, events, curfds, -1);
	
        if (nfds == -1) {
            perror("epoll_wait");
            break;
        }
        
        // 第二层循环用来处理所有发生了的事件 
        for (n = 0; n < nfds; ++n) {
	    // 如果新监测到一个SOCKET用户连接到了绑定的SOCKET端口，建立新的连接
            if (events[n].data.fd == listenfd) { // 注意，这个listen不仅仅是针对第一个连接进来的那个请求，只要是和这个IP的这个端口有关的连接，那么文件描述符listenfd都是可以达到监听是否有新连接的目的。
                len = sizeof(struct sockaddr);
                connfd = accept(listenfd, (struct sockaddr *) &their_addr, &len);
                if (connfd < 0) {
                    perror("accept");
                    continue;
                } 
                else
                    printf("有连接来自于： %s:%d， 分配的 socket 为:%d\n", inet_ntoa(their_addr.sin_addr), ntohs(their_addr.sin_port), connfd);

                setnonblocking(connfd);

                // 设置用于注册的 读操作 事件
                ev.events = EPOLLIN | EPOLLET;
                // 设置用于读操作的文件描述符
                ev.data.fd = connfd;
		
                //注册ev
                epoll_ctl(epfd, EPOLL_CTL_ADD, connfd, &ev);
                curfds ++;
            } 
            else if (events[n].events & EPOLLIN) { // 如果是已经连接的用户，并且收到数据（即此时可读），那么进行读入
                printf("EPOLLIN\n");

                if ((sockfd = events[n].data.fd) < 0)
                    continue;

                int len;
                bzero(buf, MAXBUF + 1);
		
                /* 接收客户端的消息 */
                /*len = read(sockfd, buf, MAXBUF);*/
		
                len = recv(sockfd, buf, MAXBUF, 0);
                if (len > 0)
                    printf("%d接收消息成功:'%s'，共%d个字节的数据\n", sockfd, buf, len);
                else {
                    if (len < 0) {
                        printf("消息接收失败！错误代码是%d，错误信息是'%s'\n", errno, strerror(errno));
                        epoll_ctl(epfd, EPOLL_CTL_DEL, sockfd, &ev);
                        curfds--;
                        continue;
                    }
                }

			    // 设置用于写操作的文件描述符
				ev.data.fd = sockfd;
			    // 设置用于注册的写操作事件
				ev.events = EPOLLOUT | EPOLLET;
		
                /*修改sockfd上要处理的事件为EPOLLOUT*/
		        epoll_ctl(epfd, EPOLL_CTL_MOD, sockfd, &ev); //修改标识符，等待下一个while循环时发送数据，异步处理的精髓!!!!! ?????
            } 
            else if (events[n].events & EPOLLOUT) { // 如果可以往表示连接的文件描述符中写入数据
				printf("EPOLLOUT\n"); // 打印出可写事件已经发生
				sockfd = events[n].data.fd;
				
				bzero(buf, MAXBUF + 1);
				strcpy(buf, "Server already processes!");

				send(sockfd, buf, strlen(buf), 0);

		                // 设置用于读操作的文件描述符
				ev.data.fd = sockfd;
		                // 设置用于注册的读操作事件
				ev.events = EPOLLIN | EPOLLET;
				
		                // 修改sockfd上要处理的事件为EPOLIN
				epoll_ctl(epfd, EPOLL_CTL_MOD, sockfd, &ev);
	        }
        }
    }
    
    close(listenfd);
    return 0;
}

```

对比这两种模型的服务器，我们很容易的就可以这两种方式的共同点：这两种模型都是分为两部分的，**处理与网络相关的部分和监听文件描述符上事件发生的部分**。

而这两种方式的不同点也体现在监听文件描述符上事件发生的部分上面。我们可以看出select模型它对事件监听的处理相比于epoll要更加粗糙一些，没有epoll那么详细。

还有一个不同的地方是当select、epoll返回的时候，两者的处理的方式有很大的不同。select模型它是判断一个我们想要处理的文件描述符是否在文件描述符集合里面，如何判断呢？**对整个文件描述符集合**进行遍历（这个效率还是很低的）。而epoll模型它是直接：

```c++
events[n].data.fd == listenfd
```

或者是先判断一下当前`events`是否是我们想要的那个事件，然后再来处理。

这种行为其实就是在告诉我们select和epoll的一个不同点：select返回的文件描述符集合它们不全是已经就绪好了的，我们需要通过遍历去寻找出那个我们需要的处理的文件描述符；而epoll返回的`events`数组里面所包含的文件描述符集合都是已经就绪好了的，可以直接处理了（当然，这次循环你也可以不去处理这个已经就绪好了的文件描述符）。

epoll模型中还有一个细节就是在修改文件描述符上要处理的事件的时候，并没有使用`EPOLL_CTL_ADD`参数，而是使用`EPOLL_CTL_MOD`。为什么呢？因为在内核中的红黑树上面，并没有删除掉这个文件描述符，因此还可以继续使用。

额，这次就先讲到这里，以后发现了新大陆，继续更新......

加油，happy ending......