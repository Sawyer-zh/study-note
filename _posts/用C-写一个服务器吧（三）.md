---
title: 用C++写一个服务器吧（三）
date: 2017-10-08 11:12:52
tags:
- C++
- Linux
- 网络编程
- 服务器开发
---

Hello，小伙伴们，好久没有去写我们的那个Web服务器了，原因呢是有许多事情要做，最近在刷一本700多面的书，可算是刷完了，花了快半个月了。虽然费了好些时间，但是收获满满的哦，自己对计算机底层的理解更深了一些。

好了，进入正题，这次我给大家带来的教程是为我们的Web服务器增加显示**动态网页**的功能。

### 动态网页

何为动态网页？简单来说，就是针对一个模板是一样（什么是模板一样呢？指的是页面的布局呀，js特效等等是一样的）的网页，随着请求的参数的改变，页面的内容会发生改变。

### 如何实现动态网页

那么如何根据参数的不同来实现动态网页呢？在这里，我们使用一个叫做`CGI`的东西。

### CGI

何为CGI？翻译过来就是Common Gateway Interface，即通用网关接口。熟悉计算机网络的小伙伴们应该知道那个网关是啥意思，简单的讲就是**网关是一个翻译器，一种充当转换重任的计算机系统或设备。使用在不同的通信协议、数据格式或语言，甚至体系结构完全不同的两种系统之间。**通常与网关输入输出两端通信使用的是不同的协议。**即一方是HTTP协议，另一方可能是其他协议，比如企业内部的自定义协议**（CGI程序也是差不多的原理）。然而，我们提到的计算机网络课中的网关大多指的是硬件层面上的网关。而我们这里要重点讲解的CGI可以说是软件层面上的网关。

当有一个请求过来的时候，如果它是请求一个动态的网页，那么，Web服务器可以调用CGI程序，然后这个CGI程序去处理那些从客户端传递过来的参数。

OK，小伙伴们现在对CGI应该有一个简单的认识了，如果还是不太清楚，我们跟着代码过一遍即可理解了。

首先，

我们先写一个简单的HTML页面：

```html
<!DOCTYPE html>
<html>
<head>
    <title>login page</title>
</head>
<body>
    <h1>form</h1>  
    <form action="/cgi-bin/login.cgi" method="get">  
    <table>  
        <tr>  
            <td>username: </td>  
            <td><input name="username"/></td>  
        </tr>  
            <tr>  
            <td>password: </td>  
            <td><input name="password"/></td>  
        </tr>  
        <tr>  
            <td><input type="submit" value="OK"/></td>  
        </tr>  
        </table>  
    </form>
</body>
</html>
```

然后，当我们点击提交按钮的时候，一个GET请求就会发送给我们写的Web服务器。此时我们得在Web服务器中判断一个请求是否是请求动态网页的：

```c++
bool is_cgi(char* buff) {
	char* start = buff + 5;
	char* end = strchr(start, '/');
	const char* target = "cgi-bin"; // 只读

	for (char* p = start; p != end; p++) {
        if (*p != *target) {
        	return false;
        }

        target++;
	}

	return true;
}
```

小伙伴们如果对代码不是很熟悉可以先不用搞清楚，只需要知道这些代码大概要做什么即可。

然后，我们需要获取对应的CGI程序的文件名，我们解析出来：

```c++
char* get_cgi_file_name(char* buff) {
	char* file_name = new char[BUFFSIZE];
    file_name = buff + 13;
    char* end = strchr(file_name, ' ');
    *end = '\0';
    cout<<file_name<<endl;
    cout<<strlen(file_name)<<endl;

    return file_name;
}
```

OK，此时，我们已经获取了CGI程序的文件名了，我们需要执行它。然而，这里会出现一个问题，什么问题呢？就是主函数只能有一个的问题。那么我们怎么去执行那个CGI程序呢？对的，现在是并发的时代了，我们切换一个进程即可做到，换个装逼一点的词汇就是让CPU去**调度**CGI进程。

```c++
if (is_cgi(buff)) {
    char* cgi_file_name = get_cgi_file_name(buff);

    pid_t pid;
    pid = fork(); // 创建子进程，在子进程里执行CGI程序

    if (0 == pid) {
        char path[] = "/codedir/cppCode/cgi-bin/";
        strcat(path, cgi_file_name);
        printf("%s\n", path);
        execl(path, cgi_file_name, NULL);
    }
}
```

好的，接下来我们来完成一下CGI程序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

using namespace std;

int main(int argc, char const *argv[])
{
	char *buf, *p;
	char arg1[1024], arg2[1024], content[1024];

    printf("%s\n", "it is login.cgi");
    
    printf("HTTP/1.0 200 OK\r\n");
    printf("Content-type: text/html\r\n\r\n");
    printf("I am hht\n");
    
	exit(0);
}
```

接下来我们测试一下我们写的CGI程序：

![](http://oklbfi1yj.bkt.clouddn.com/%E3%80%8A%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%B8%89%EF%BC%89%E3%80%8B/1.gif)

OK，没毛病😳。CGI程序成功的被我们调用。接下来，就要利用这个CGI程序实现动态网页。

### 获取参数

那么，如果实现动态网页的话，很关键的一点就是要获取从客户端传来的参数。那么如何获取呢？

当客户端通过get或post方法向CGI程序提交了数据以后，我们可以环境变量来获取那些参数。在这里，我们通过`QUERY_STRING`环境变量可以得到**查询字符串**。（正是因为CGI这个接口协议，这些环境变量是属于该接口协议的内容）

我们通过C语言里面的`getenv()`函数来获取查询字符串。

```c
if ((buf = getenv("QUERY_STRING")) != NULL) {
    p = strchr(buf, '&');
    *p = '\0';

    strcpy(arg1, buf);
    strcpy(arg2, p + 1);

    sprintf(content, "your name is %s\n", arg1);
    sprintf(content, "your password is %s\n", arg2);

    printf("HTTP/1.0 200 OK\r\n");
    printf("Content-type: text/html\r\n\r\n");
    printf("%s\n", content);

    fflush(stdout);
}
```

OK，我们现在来看看能否解析出我们传递给Web服务器的参数（注意，**在这里我们只能传递两个参数**）。这个操作我们在浏览器里面进行：

![](http://oklbfi1yj.bkt.clouddn.com/%E3%80%8A%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%B8%89%EF%BC%89%E3%80%8B/2.gif)

咦( ′◔ ‸◔`)，咋没有显示呢？我们来找找bug。







