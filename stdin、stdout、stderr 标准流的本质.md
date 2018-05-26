# stdout、stderr 标准流的本质

默认向屏幕输出

**每个进程拥有独立的内存空间**，同时**每个进程来也拥有独立的 fd 号码**，一般 stdin, stdout, stderr 的 fd 号码是 0, 1, 2，如果之后打开了其它的文件，文件句柄号会累加。

简单的说，他们所指向的文件句柄的值是 0, 1, 2 

总结一下：
对于应用层来说，**stdin / stdout / stderr 实际上就是在程序开始运行时被默认打开的文件而已，跟你自己用 fopen()/open() 去打开一个文件没有区别**，而有区别的地方在于--无论是 tty0 ~ tty6 也好，还是 /pts/0 等等也好，**它们不是普通的文件，而是设备文件**-- Linux 宣称一切皆文件，因为在底层上 Linux 把文件和设备统一起来用 inode 来管理

标准输出(设备)文件，对应终端的屏幕。进程将从标准输入文件中得到输入数据，将正常输出数据输出到标准输出文件，而将错误信息送到标准错误文件中。在C中，程序执行时，一直处于开启状态

stderr -- 标准错误输出设备

stdout -- 标准输出设备 (printf("..")) 同 stdout

**两者默认向屏幕输出**

但如果用转向标准输出到磁盘文件，则可看出两者区别。stdout输出到磁盘文件，stderr在屏幕

###  STDIN_FILENO的作用及与stdin 的区别

1、数据类型不一致：
stdin类型为 `FILE*`
STDIN_FILENO类型为 `int`
使用stdin的函数主要有：fread、fwrite、fclose等，基本上都以f开头
使用STDIN_FILENO的函数有：read、write、close等

2、stdin等是`FILE *`类型，属于标准I/O，高级的输入输出函数。在`<stdio.h>`。
`STDIN_FILENO`等是文件描述符，是**非负整数**，一般定义为 0, 1, 2（STDIN_FILENO 是0），**属于没有buffer的I/O**，直接调用系统调用，在`<unistd.h>`









