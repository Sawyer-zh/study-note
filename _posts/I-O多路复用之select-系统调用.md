---
title: I/O多路复用之select()系统调用
date: 2017-09-13 14:15:44
tags:
- C语言
- Linux
---

### 为什么要使用I/O多路复用

应用程序常常需要在多于一个文件描述符上阻塞：例如响应键盘输入（stdin）、进程间通信以及同时操作多个文件。

在不使用线程（怎么理解线程的存在呢？我么可以举一个例子。当我们运行qq这个进程的时候，是可以执行不同的任务的。例如，我们可以在使用qq发送消息的同时来收发文件。而这两个不同的任务就是利用线程来完成的），**尤其是独立处理每一个文件的情况下，进程无法在多个文件描述符上同时阻塞。如果文件都处于准备好被读写的状态，同时操作多个文件描述符是没有问题的**。但是，一旦在该过程中出现一个未准备好的文件描述符（就是说，如果一个`read()`被调用，但没有读入数据），则这个进程将会阻塞，不能再操作其他文件。可能阻塞只有几秒钟，但是应用无响应也会造成不好的用户体验。然而，如果文件描述符始终没有任何可用数据，就可能一直阻塞下去。

**如果使用非阻塞I/O，应用可以发起I/O请求并返回一个特别的错误，从而避免阻塞**。但是，从两个方面来讲，这种方法效率较差。首先，进程需要以某种不确定的方式**不断发起I/O操作**，直到某个打开的文件描述符准备好进行I/O。其次，如果程序可以睡眠的话将更加有效，可以让处理器进行其他工作，直到一个或更多文件描述符可以进行I/O时再唤醒。

### 三种I/O多路复用方案

I/O多路复用允许应用在多个文件描述符上同时阻塞，并在其中某个可以读写时收到通知。这时I/O多路复用就成了应用的关键所在。

I/O多路复用的设计遵循一下原则：

1、I/O多路复用：当任何文件描述符准备好I/O时告诉我

2、在一个或更多文件描述符就绪前始终处于睡眠状态

3、唤醒：哪个准备好了？

4、在不阻塞的情况下处理所有I/O就绪的文件描述符

5、返回第一步，重新开始

Linux提供了三种I/O多路复用方案：`select`、`poll`、`epoll`。

### 先来说说select()

#### select()系统调用的声明

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

/*
作用：通知执行了select()的进程哪个Socket或文件可读
返回值：负值：select错误，见ERRORS。 
       正值：某些文件可读写或出错  
       0：等待超时，没有可读写或错误的文件
*/

int select (int n,
            fd_set *readfds,
            fd_set *writefds,
            fd_set *exceptfds,
            struct timeval *timeout
            );
```

其中`fd_set`是`select`机制中提供的一种数据结构，实际上是一`long`类型的数组，每一个数组元素都能与一打开的文件句柄（不仅是`socket`句柄，还是其他文件或命名管道或设备句柄）建立联系，建立联系的工作由程序员完成，当调用`select()`时，由内核根据IO状态修改`fd_set`的内容，由此来通知执行了`select()`的进程哪一个`socket`或文件发生了可读或可写事件

监测的文件描述符可以分为三类，分别等待不同的事件。监测`readfds`集合中的文件描述符，确认其中是否有可读数据（也就是说，确认好了的文件描述符的读操作可以**无阻塞**的完成）。监测`writefds`集合中的文件描述符，**确认**其中是否有一个写操作可以**不阻塞**地完成。监测`exceptfds`中的文件描述符，确认其中是否有出现异常发生或者出现带外数据（这种情况只适用于套接字）。指定的集合可能为空（NULL）。相应的，`select()`则不对此类事件进行监测。

**成功返回时，每个集合只包含对应类型的I/O就绪的文件描述符**。举个例子，`readfds`集合中有两个文件描述符：7和9.当调用返回时，如果7还在集合中，该文件描述符就准备好进行无阻塞I/O了。如果9已不在集合中，它可能在被读取时会发生阻塞。**出现错误返回-1**。

第一个参数n，等于**所有集合中文件描述符的最大值加1**。这样，`select()`的调用者需要找到最大的文件描述符值，并将其加1后传给第一个参数。

`timeout`参数是一个指向`timeval`结构体的指针，定义如下：

```c
#include <sys/time.h>
struct timeval {
    long tv_sec; /* seconds */
    long tv_usec; /* microseconds */
};
```

如果这个参数不是NULL，即使此时没有文件描述符处于I/O就绪状态，`select()`调用也将在`tv_sec秒、tv_usec微秒`后返回。即 **select在timeout时间内阻塞，超时时间之内有事件到来就返回了，否则在超时后不管怎样一定返回**。

**如果时限中的两个值都是0**，调用会立即返回，并报告调用时所有事件对应的文件描述符均不可用，且不等待任何后续事件。

若将**NULL**以形参传入，即不传入时间结构，就是将select**置于阻塞状态，**一定等到监视文件描述符集合中某个文件描述符发生变化为止

### 举个select()的小例子

```c
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

#define TIMEOUT 5 /* 选择超时秒数 */
#define BUF_LEN 1024 /* 读缓冲区字节 */

int main(int argc, char const *argv[])
{
    struct timeval tv;
    fd_set readfds;
    int ret;
     
    /* 等待输入 */
    FD_ZERO(&readfds); // 把writefds集合中的所有文件描述符移除
    FD_SET(STDIN_FILENO, &readfds); // 向writefds集合中添加文件描述符STDIN_FILENO。STDIN_FILENO就是标准输入设备（一般是键盘）的文件描述符。它的值为0

    /* 设置等待为5秒 */
    tv.tv_sec = TIMEOUT;
    tv.tv_usec = 0;

    /* 在指定的tv时间内阻塞 */
    ret = select(STDIN_FILENO + 1, &readfds, NULL, NULL, &tv); // 通知执行了select()的进程哪个Socket或文件可读

    if (ret == -1) {
    	perror("select");
    	return 1;
    }
    else if (!ret) {
    	printf("%d 秒 已经过去了. \n", TIMEOUT);
    	return 0;
    }

    if (FD_ISSET(STDIN_FILENO, &readfds)) { // 测试给定的文件描述符在不在给定的集合中。检查fdset联系的文件句柄fd是否可读写，当>0表示可读写
    	char buf[BUF_LEN + 1];
    	int len;
    	/* 保证没有阻塞 */
    	len = read(STDIN_FILENO, buf, BUF_LEN);
    	if (len == -1) {
    		perror("read");
    		return 1;
    	}
    	if (len) {
    		buf[len] = '\0';
    		printf("read: %s\n", buf);
    	}
    	return 0;
    }
    fprintf(stderr, "This should not happen!\n");

	return 0;
}
```

让我们执行这段代码之后，等待5秒：

![](http://oklbfi1yj.bkt.clouddn.com/I/O%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E4%B9%8Bselect%28%29%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8/1.gif)

可以看到，在这5秒内，进程是处于阻塞状态的（因为文件描述符的状态没有发生变化）

现在让我们执行完这段代码后输入一些内容：

![](http://oklbfi1yj.bkt.clouddn.com/I/O%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E4%B9%8Bselect%28%29%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8/2.gif)

上面这个例子虽然只是检测了一个文件描述符（因此不是多路复用），但是对于`select()`这个系统调用的用法已经很清晰了。





