# 阻塞和非阻塞read/write

**如果被阻塞了，操作系统可以调度别的进程**。

## read()函数

```c++
/*
返回值：若成功则返回读到的字节数，若已到文件结则返回0，若出错则返回-1并设置errno  
*/
#include<unistd.h>
ssize_t read(int filedes, void *buf,size_t nbytes);
```

参数`nbytes`是请求读取的字节数，读上来的数据保存在缓冲区`buf`中（即内存相对于磁盘来说是它的缓冲区），同时文件的当前读写位置向后移。注意**这个读写位置和使用C标准I/O库时的读写位置有可能不同，这个读写位置是记在内核中的，而使用C标准I/O库时的读写位置是用户空间I/O缓冲区中的位置**。**比如用`fgetc`读一个字节，`fgetc`有可能从内核中预读1024个字节到I/O缓冲区中，再返回第一个字节，这时该文件在内核中记录的读写位置是1024，而在`FILE`结构体中记录的读写位置是1**。

## write()函数

```c++
/*
返回值：若成功则返回已写的字节数，若出错则返回-1并设置errno  
*/
#include<unistd.h>
ssize_t write(int filedes,const voidvoid *buf,size_t nbytes);
```

写常规文件时，`write`的返回值通常等于请求写的字节数`nbytes`，而**向终端设备或网络写则不一定**。

## 阻塞与非阻塞的情况

**读常规文件是不会阻塞的，不管读多少字节，`read`一定会在有限的时间内返回（也就是说，即使没有读到我们想要读到的字节数大小，也会返回）**。**从终端设备或网络读则不一定，如果从终端输入的数据没有换行符，调用`read`读终端设备就会阻塞，如果网络上没有接收到数据包，调用`read`从网络读就会阻塞**，至于会阻塞多长时间也是不确定的，如果一直没有数据到达就一直阻塞在那里。同样，**写常规文件是不会阻塞的（因为，无论有没有全部写入，也会返回，而不会一直阻塞在那里啥也不干），而向终端设备或网络写则不一定**。

## 阻塞读终端的一个例子

```c++
#include <unistd.h>  
#include <stdlib.h>  
  
int main(void)  
{  
    char buf[10];  
    int n;  
    n = read(STDIN_FILENO, buf, 10);  
    if (n < 0) {  
        perror("read STDIN_FILENO");  
        exit(1);  
    }  
    write(STDOUT_FILENO, buf, n);  
    return 0;  
}  
```

执行结果：

```shell
[root@localhost apue2]# ./a.out  
hello  
hello  
[root@localhost apue2]# ./a.out   
hello world  
hello worl[root@localhost apue2]# d  
bash: d: command not found  
[root@localhost apue2]#   
```

第一次执行`a.out`的结果很正常（虽然，打算从标准输入读10个字节，但是，从终端只输入了6个字节（包括那个回车），所以不会阻塞，直接返回），而第二次执行的过程有点特殊，现在分析一下：

1、**Shell进程创建`a.out`进程**，`a.out`进程开始执行，而Shell进程睡眠等待`a.out`进程退出。

2、`a.out`调用`read`时睡眠等待（此时处于阻塞状态（因为没有遇到换行符）），直到终端设备输入了换行符才从`read`返回，`read`只读走10个字符，**剩下的字符仍然保存在内核的终端设备输入缓冲区中**。

3、`a.out`进程打印并退出，这时Shell进程恢复运行，**Shell继续从终端读取用户输入的命令，于是读走了终端设备输入缓冲区中剩下的字符d和换行符，把它当成一条命令解释执行，结果发现执行不了，没有d这个命令**。

如果在`open`一个设备时指定了`O_NONBLOCK`标志，`read`/`write`就不会阻塞。以`read`为例，如果设备暂时没有数据可读就返回-1，同时置`errno`为`EWOULDBLOCK`（或者`EAGAIN`，这两个宏定义的值相同），表示本来应该阻塞在这里（would block，虚拟语气），事实上并没有阻塞而是直接返回错误，调用者应该试着再读一次（again）。这种行为方式称为**轮询**（Poll），**调用者只是查询一下，而不是阻塞在这里死等，这样可以同时监视多个设备**：

```c++
while(1) {  
    非阻塞read(设备1);  
    if(设备1有数据到达)  
        处理数据;  
    非阻塞read(设备2);  
    if(设备2有数据到达)  
        处理数据;  
    ...  
} 
```

如果`read(设备1)`是阻塞的，那么只要设备1没有数据到达就会一直阻塞在设备1的`read`调用上，即使设备2有数据到达也不能处理，使用非阻塞I/O就可以避免设备2得不到及时处理。

非阻塞I/O有一个缺点，如果所有设备都一直没有数据到达，调用者（进程）需要反复查询做无用功，**如果阻塞在那里，操作系统可以调度别的进程执行，就不会做无用功了**。在使用非阻塞I/O时，通常不会在一个`while`循环中一直不停地查询（这称为Tight Loop），而是每延迟等待一会儿（sleep）来查询一下，以免做太多无用功，**在延迟等待的时候可以调度其它进程执行**（因为此时的进程会被挂起）：

```c++
while(1) {  
    非阻塞read(设备1);  
    if(设备1有数据到达)  
        处理数据;  
    非阻塞read(设备2);  
    if(设备2有数据到达)  
        处理数据;  
    ...  
    sleep(n);  
} 
```

这样做的问题是，**设备1有数据到达时可能不能及时处理，最长需延迟n秒才能处理，而且反复查询还是做了很多无用功**。而`select()`函数**可以阻塞地同时监视多个设备**，还可以设定阻塞等待的超时时间，从而圆满地解决了这个问题。

## 非阻塞读终端

以下是一个非阻塞I/O的例子。目前我们学过的可能引起阻塞的设备只有终端，所以我们用终端来做这个实验。**程序开始执行时在0、1、2文件描述符上自动打开的文件就是终端**，**但是没有`O_NONBLOCK`标志**。读标准输入是阻塞的。我们可以重新打开一遍设备文件`/dev/tty`（表示当前终端），在打开时指定`O_NONBLOCK`标志。

```c++
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <string.h>  
#include <stdlib.h>  
  
#define MSG_TRY "try again\n"  
  
int main(void)  
{  
    char buf[10];  
    int fd, n;  
    fd = open("/dev/tty", O_RDONLY|O_NONBLOCK);  
    if(fd<0) {  
        perror("open /dev/tty");  
        exit(1);  
    }  
tryagain:  
    n = read(fd, buf, 10);  
    if (n < 0) {  
        if (errno == EAGAIN) {  
            sleep(1);  
            write(STDOUT_FILENO, MSG_TRY, strlen(MSG_TRY));  
            goto tryagain;  
        }     
        perror("read /dev/tty");  
        exit(1);  
    }  
    write(STDOUT_FILENO, buf, n);  
    close(fd);  
    return 0;  
}  
```

























