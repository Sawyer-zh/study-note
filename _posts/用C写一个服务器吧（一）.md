---
title: 用C++写一个Web服务器吧（一）
date: 2017-09-08 16:49:19
tags:
- C++
- Linux
- 网络编程
- 服务器开发
---

[获取本服务器完整的源码](https://github.com/huanghantao/HHTWS)

首先，要实现一个Web服务器，必须要有一定的理论基础，才知道要做什么事情（如何写代码）

### 理论知识

首先，我们要知道什么是客户端和服务器

#### 客户端

> 指**与服务器相对应**，为客户提供本地服务的程序。除了一些只在本地运行的应用程序之外，一般安装在普通的客户机上，需要与服务端互相配合运行
>
> ​                                                                                                      --摘自百度百科

举个例子，我们常常用的**浏览器就是一个客户端**

#### 服务器

与客户端相对的，提供服务的。一般服务器上面都放有客户端需要的资源。当客户端请求服务器上的某些资源的时候，如果服务器允许的话，那么服务器可以返回这些资源给客户端。例如，浏览器去访问`www.baidu.com`的时候，百度的服务器就会返回一个`html`文件给浏览器，然后经过浏览器的渲染，呈现给用户一个美丽的页面

#### 客户端与服务器进行交流的本质

我们知道，无论是客户端的程序还是服务器的程序，本质都是程序。那么当程序跑起来之后，就是进程了。而进程之间进行交流就需要使用进程通信的一些方法了。并且，这两个进程是处于不同的位置对吧（可能在世界的两端），不是简单的由父进程`fork`一下，然后使用类似于管道、信号之类的进程通信手段就可以完成通信的。既然它们是在网络中的，就需要遵循**网络协议**

#### 网络协议

HTTP是一个客户端和服务器端请求和应答的标准协议。通常，由HTTP客户端发起一个请求，建立一个到服务器指定端口（默认是80端口）的TCP连接

因此我们要做的工作就是利用Linux系统提供的TCP通信接口来实现HTTP协议。此时我们用到的就是**socket（套接字）**

**每一对网络连接称为一个socket对，包括两个端点的socket地址**

#### socket处理请求与响应示意图

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/9.PNG)

我们接下来要写的代码就是**围绕这幅图进行的**

#### IP和端口号

IP是用来在互联网中寻找主机用的，端口号则是用来区分应用的。比如说，我要访问`www.baidu.com`这个网页，我是需要知道它的IP地址才能访问的，除此之外，我还必须知道端口号。为什么呢？我们可以把IP地址想象成**家**，而我不可能和**家**通信吧，我还必须知道我要通信的这个人在哪里对吧，此时端口号就起了作用，用来标识**哪个人**

### 实现接受GET请求的功能

#### 代码实现

```c
#include <iostream>
#include <string>
#include <cstring>
#include <cstdlib>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/unistd.h>
#include <netinet/in.h>
#include <fstream>

using namespace std;

const int BUFFSIZE = 1024;
const int MAXLINK = 10; // 未经过处理的连接请求队列可以容纳的最大数目
const int DEFAULT_PORT = 8080;

char* get_file_name (char* buff) {
    char* file_name = buff + 5;
    char *space = strchr(file_name, ' ');
    *space = '\0';
    return file_name;
}
 
void deal_get_http(int connect_fd, char* buff) {
    char* file_name = get_file_name(buff);
    const char http_correct_header[] = "HTTP/1.1 200 OK\r\nContent-type: text/html\r\n\r\n";
    int res = write(connect_fd, http_correct_header, strlen(http_correct_header));
    if (res > 0) {
    	cout<<"send success"<<endl;
    }
}

bool is_get_http(char* buff) {
    if (!strncmp(buff, "GET", 3)) { // 如果是GET请求
    	return true;
    }
    else {
    	return false;
    }
}

int main(int argc, char const *argv[])
{


    int socket_fd, connect_fd;
    struct sockaddr_in servaddr;
    char buff[BUFFSIZE];

    socket_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (socket_fd == -1) {
            cout<<"create socket error"<<endl;
            return -1;
    }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY); // IP必须是网络字节序，INADDR_ANY是绑定本机上所有IP
    servaddr.sin_port = htons(DEFAULT_PORT); // 端口号必须是网络字节序
    
    if (bind(socket_fd, (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1) {
    	cout<<"bind error"<<endl;
    	return -1;
    }

    if (listen(socket_fd, MAXLINK) == -1) {
        cout<<"listen error"<<endl;
    }
    
    connect_fd = accept(socket_fd, (struct sockaddr*)NULL, NULL);
    if (connect_fd == -1) {
    	cout<<"accept error"<<endl;
    }
    else {
    	cout<<"连接成功"<<endl;
    }

	memset(buff, '\0', sizeof(buff));

	recv(connect_fd, buff, BUFFSIZE - 1, 0); // 把请求头（或发送的消息）写入buff中
	send(connect_fd, buff, BUFFSIZE - 1, 0); // 向客户端发送消息（发送buff中的内容）

	cout<<"recive message from client: "<<buff<<endl;

	if (is_get_http(buff)) {
		cout<<"it is get http"<<endl;
		deal_get_http(connect_fd, buff);
	}

    close(connect_fd);  
    close(socket_fd);

    return 0;
}
```

好的，到这里，我们就已经写完了一个可以接收GET请求的服务器了，我们来测试一下：

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7/1.gif)

在上面，我们只是模拟了使用telnet发送一个GET请求，这个请求去请求`index.html`文件，然后我们返回一部分响应头给客户端，此时并没有返回`index.html`的内容给客户端。接下来，我们就来完成这部分功能，一步一步的让这个服务器健全起来。

### 返回请求文件的内容

#### 代码实现

```c++
void deal_get_http(int connect_fd, char* request_header) {
    char* file_name = get_file_name(request_header);
    int file_size = get_file_size(file_name);
    char file_type[BUFFSIZE];
    get_filetype(file_name, file_type);

    int fd = open(file_name, O_RDONLY);
    void* file_in_mem_addr = mmap(0, file_size, PROT_READ, MAP_PRIVATE, fd, 0); // 存储映射
    close(fd);

    char buff[BUFFSIZE];
    strcat(buff, "HTTP/1.0 200 OK\r\n");
    strcat(buff, "Server: HHTWS Web Server\r\n");
    strcat(buff, "Connection: close\r\n");
    strcat(buff, "Content-length: \r\n");
    strcat(buff, "Content-type: \r\n\r\n");
    
    send(connect_fd, buff, strlen(buff), 0);
    send(connect_fd, file_in_mem_addr, file_size, 0);

    munmap(file_in_mem_addr, file_size);
}
```

我们来看看效果：

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7/2.gif)

#### 存储映射机制

除了标准文件I/O，内核提供了另一种高级的I/O方式，允许应用程序**将文件映射到内存中**，即**内存和文件中数据是一一对应的**。程序员可以**直接通过内存来访问文件，就像操作内存的数据块一样**，甚至可以写入内存数据区，然后通过透明的映射机制将文件写入磁盘。

当映射一个文件描述符的时候，描述符引用计数增加。**如果映射文件后关闭文件，你的进程依然可以访问该文件**。当你取消映射或者进程终止时，对应的文件引用计数会减1。

#### 遇到的问题

##### 读取整个文件

开始的时候是想到了常规的方法，也就是使用`fgets()`函数等等，但是，因为它会给每一个字符串加上`'\0'`。所以当我使用它的时候，出现了一些乱码。之后我突然想起了我最近在《Linux系统编程》中看到的一个存储映射机制，于是就用上了它。非常方便的把文件的内容获取到了。

##### Segmentation fault问题

我在运行这个服务器的时候，出现了这个错误。最后经过排查，发现是由于存在野指针。开始的时候，我是这样定义一个文件类型的：

```c++
char* file_type;
```

然后我就直接把这个野指针给传进`get_filetype`函数里面去了。导致问题发生。

好了，到这里，我们就写完了一个可以说是完整的支持GET请求的Web服务器了。那小伙伴们是不是想在浏览器里面试一试效果！！看看能不能看到这个HTML文件被浏览器渲染出来！！

我们来试试！！

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7/3.gif)

咦( ′◔ ‸◔`)？为什么不能在浏览器上面看到呢？小伙伴们，我将在下面一篇文章中继续为大家讲解。happy ending

（未完--持续更新加入新功能中）

















