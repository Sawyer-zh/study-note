---
title: 使用signal系统调用处理进程接收到的信号
date: 2017-08-26 19:12:17
tags:
- 操作系统
- 网络编程
- C语言
---

我们知道，一个进程在接收到一个信号之后，一般都会中断（例如，当进程收到了`SIGINT`信号之后，会终止运行的程序）。然而，我们可以使用`signal()`这个系统调用来处理接收到的信号

`signal()`函数有两个参数。其中，第一个参数指明了所要处理的信号类型（例如：`SIGINT`），它可以取**除了 SIGKILL 和 SIGSTOP 外**的任何一种信号。第二个参数描述了与信号关联的动作，一般是一个**函数地址**

举个例子：

```c
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>

int catch(int sig);

int main(void)
{
    signal(SIGINT, catch); /* 将 SIGINT 信号与 catch 函数关联 */
    printf("in main function\n");
    sleep(10);
    printf("end\n");
    return 0;
}

int catch(int sig)
{
    printf("Catch succeed!\n");
    return 1;
}
```

我们编译它：

![](https://segmentfault.com/img/bVTIOn?w=827&h=203)



报错：说第二参数需要一个`__sighandler_t`类型的参数

遇到这种情况该如何下手？

我们如果不是很清楚`signal`的原型究竟是什么，可以使用`man`命令来查看（`man`命令不仅仅可以用来查看命令，还可以用来查看函数）：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8signal%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E5%A4%84%E7%90%86%E8%BF%9B%E7%A8%8B%E6%8E%A5%E6%94%B6%E5%88%B0%E7%9A%84%E4%BF%A1%E5%8F%B7/2.PNG)

我们可以看到那个`void`，所以，把`catch`函数的返回值类型改为`void`：

```c
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>

void catch(int sig);

int main(void)
{
    signal(SIGINT, catch); /* 将 SIGINT 信号与 catch 函数关联 */
    printf("in main function\n");
    sleep(10);
    printf("end\n");
    return 0;
}

void catch(int sig)
{
    printf("Catch succeed!\n");
    return 1;
}
```

再次编译，然后运行：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8signal%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E5%A4%84%E7%90%86%E8%BF%9B%E7%A8%8B%E6%8E%A5%E6%94%B6%E5%88%B0%E7%9A%84%E4%BF%A1%E5%8F%B7/3.PNG)

然后按下`Ctrl+c`，向这个进程发送`SIGINT`信号：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8signal%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E5%A4%84%E7%90%86%E8%BF%9B%E7%A8%8B%E6%8E%A5%E6%94%B6%E5%88%B0%E7%9A%84%E4%BF%A1%E5%8F%B7/4.PNG)



