# Linux进程间通信——使用共享内存

## 什么是共享内存

共享内存就是允许两个不相关的进程访问同一个逻辑内存。不同进程之间共享的内存通常安排为同一段物理内存。**进程可以将同一段共享内存连接到它们自己的地址空间中，所有进程都可以访问共享内存中的地址**，就好像它们是由用C语言函数malloc分配的内存一样。而**如果某个进程向共享内存写入数据，所做的改动将立即影响到可以访问同一段共享内存的任何其他进程**。

**共享内存并未提供同步机制**。所以我们通常需要用其他的机制来同步对共享内存的访问，例如前面说到的信号量。

## 共享内存的使用

### shmget函数

该函数用来创建共享内存，它的原型为：

```c++
int shmget(key_t key, size_t size, int shmflg);
```

第一个参数，与信号量的semget函数一样，程序需要**提供一个参数key**（非0整数），它有效地**为共享内存段命名**，**shmget函数成功时返回一个与key相关的共享内存标识符**（非负整数），用于后续的共享内存函数。调用失败返回-1。

不相关的进程可以通过该函数的返回值访问同一共享内存，它代表程序可能要使用的某个资源，程序对所有共享内存的访问都是间接的，程序先通过调用shmget函数并提供一个键，再由系统生成一个相应的共享内存标识符（shmget函数的返回值），只有shmget函数才直接使用信号量键，所有其他的信号量函数使用由semget函数返回的信号量标识符。

第二个参数，size**以字节为单位指定需要共享的内存容量**。

第三个参数，**shmflg是权限标志**，它的作用与open函数的mode参数一样，如果要想在key标识的共享内存不存在时，创建它的话，可以与IPC_CREAT做或操作。**共享内存的权限标志与文件的读写权限一样**，举例来说，**0644，它表示允许一个进程创建的共享内存被内存创建者所拥有的进程向共享内存读取和写入数据，同时其他用户创建的进程只能读取共享内存**。

### shmat函数

第一次创建完共享内存时，它还不能被任何进程访问，shmat函数的作用就是用来启动对该共享内存的访问，并把共享内存连接到当前进程的地址空间。它的原型如下：

```c++
void* shmat(int shm_id, const void *shm_addr, int shmflg);
```

第一个参数，shm_id是由shmget函数返回的共享内存标识。

第二个参数，shm_addr指定共享内存连接到当前进程中的地址位置，**通常为空（0或者NULL），表示让系统来选择共享内存的地址**。

第三个参数，shm_flg是一组标志位，**通常为0**。

调用成功时**返回一个指向共享内存第一个字节的指针**，如果调用失败返回-1。

也就是说，必须要先使用shmat函数申请对共享内存的访问，之后才能访问共享内存。

### shmdt函数

该函数用于**将共享内存从当前进程中分离**。注意，将共享内存分离并不是删除共享内存，只是使该共享内存对当前进程不再可用。它的原型如下：

```c++
int shmdt(const void* shmaddr);
```

参数shmaddr是shmat函数返回的地址指针，调用成功时返回0，失败时返回-1。

### shmctl函数

与信号量的semctl函数一样，用来控制共享内存，它的原型如下：

```c++
int shmctl(int shm_id, int command, struct shmid_ds* buf);
```

第一个参数，shm_id是shmget函数返回的共享内存标识符。

第二个参数，command是要采取的操作，它可以取下面的三个值 ：

​    IPC_STAT：把shmid_ds结构中的数据设置为共享内存的当前关联值，即用共享内存的当前关联值覆盖shmid_ds的值。

​    IPC_SET：如果进程有足够的权限，就把共享内存的当前关联值设置为shmid_ds结构中给出的值

​    IPC_RMID：删除共享内存段

第三个参数，buf是一个结构指针，它指向共享内存模式和访问权限的结构。

shmid_ds结构至少包括以下成员：

```c++
struct shmid_ds  
{  
    uid_t shm_perm.uid;  
    uid_t shm_perm.gid;  
    mode_t shm_perm.mode;  
};
```

## 使用共享内存进行进程间通信

下面就以两个不相关的进程来说明进程间如何通过共享内存来进行通信。其中一个文件shmread.c创建共享内存，并读取其中的信息，另一个文件shmwrite.c向共享内存中写入数据。为了方便操作和数据结构的统一，为这两个文件定义了相同的数据结构，定义在文件shmdata.h中。结构shared_use_st中的written作为一个可读或可写的标志，非0：表示可读，0表示可写，text则是内存中的文件。

### shmdata.h

shmdata.h的源代码如下：

```c++
#ifndef _SHMDATA_H_HEADER
#define _SHMDATA_H_HEADER
  
#define TEXT_SZ 2048
  
struct shared_use_st
{
    int written;//作为一个标志，非0：表示等待读取，0：表示可写
    char text[TEXT_SZ];//记录写入和读取的文本
};
  
#endif
```

### shmread.c

源文件shmread.c的源代码如下：

```c++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/shm.h>
#include "shmdata.h"

int main()
{
	int running = 1; //程序是否继续运行的标志
	void* shm = NULL; //分配的共享内存的原始首地址
	struct shared_use_st *shared = NULL; // 指向shm
	int shmid;//共享内存标识符
	//创建共享内存
	shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
	if(shmid == -1)
	{
		fprintf(stderr, "shmget failed\n");
		exit(EXIT_FAILURE);
	}
	//将共享内存连接到当前进程的地址空间
	shm = shmat(shmid, 0, 0);
	if(shm == (void*)-1)
	{
		fprintf(stderr, "shmat failed\n");
		exit(EXIT_FAILURE);
	}
	printf("\nMemory attached at %X\n", (int)shm);
	//设置共享内存
	shared = (struct shared_use_st*)shm;
	shared->written = 0;
	while(running)//读取共享内存中的数据
	{
		//没有进程向共享内存写数据，所以可读取
		if(shared->written != 0)
		{
			printf("You wrote: %s", shared->text);
			sleep(rand() % 3);
			//读取完数据，设置written使共享内存段可写
			shared->written = 0; // 读取完了之后，把written设置成了0之后，write进程就可以写了（因为是共享内存，所以，read进程设置了written，会反应到write进程那里）
			//输入了end，退出循环（程序）
			if(strncmp(shared->text, "end", 3) == 0)
				running = 0;
		}
		else//有其他进程在写数据，不能读取数据
			sleep(1);
	}
	//把共享内存从当前进程中分离
	if(shmdt(shm) == -1)
	{
		fprintf(stderr, "shmdt failed\n");
		exit(EXIT_FAILURE);
	}
	//删除共享内存
	if(shmctl(shmid, IPC_RMID, 0) == -1)
	{
		fprintf(stderr, "shmctl(IPC_RMID) failed\n");
		exit(EXIT_FAILURE);
	}
	exit(EXIT_SUCCESS);
}
```

### shmwrite.c

源文件shmwrite.c的源代码如下：

```c++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>
#include "shmdata.h"

int main()
{
	int running = 1; // 程序是否继续运行的标志
	void *shm = NULL; // 分配的共享内存的原始首地址
	struct shared_use_st *shared = NULL; // 指向shm
	char buffer[BUFSIZ + 1]; // 用于保存输入的文本
	int shmid;
	//创建共享内存
	shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT); // 因为这里的key和read进程的key是一样的，所以得到的是同一个共享内存
	if(shmid == -1)
	{
		fprintf(stderr, "shmget failed\n");
		exit(EXIT_FAILURE);
	}
	//将共享内存连接到当前进程的地址空间
	shm = shmat(shmid, (void*)0, 0);
	if(shm == (void*)-1)
	{
		fprintf(stderr, "shmat failed\n");
		exit(EXIT_FAILURE);
	}
	printf("Memory attached at %X\n", (int)shm);
	//设置共享内存
	shared = (struct shared_use_st*)shm;
	while(running)//向共享内存中写数据
	{
		//数据还没有被读取，则等待数据被读取, 不能向共享内存中写入文本
		while(shared->written == 1)
		{
			sleep(1);
			printf("Waiting...\n");
		}
		//向共享内存中写入数据
		printf("Enter some text: ");
		fgets(buffer, BUFSIZ, stdin);
		strncpy(shared->text, buffer, TEXT_SZ);
		//写完数据，设置written，使共享内存中的数据等待被读取
		shared->written = 1;
		//输入了end，退出循环（程序）
		if(strncmp(buffer, "end", 3) == 0)
			running = 0;
	}
	//把共享内存从当前进程中分离
	if(shmdt(shm) == -1)
	{
		fprintf(stderr, "shmdt failed\n");
		exit(EXIT_FAILURE);
	}
	sleep(2);
	exit(EXIT_SUCCESS);
}
```

执行：

```shell
gcc -o shmread.exe shmread.c
gcc -o shmwrite.exe shmwrite.c
./shmread.exe &
./shmwrite.exe
```

分析：

1、程序shmread创建共享内存，然后将它连接到自己的地址空间。在共享内存的开始处使用了一个结构struct_use_st。该结构中有个标志written，当共享内存中有其他进程向它写入数据时，共享内存中的written被设置为0，程序等待。当它不为0时，表示没有进程对共享内存写入数据，程序就从共享内存中读取数据并输出，然后重置设置共享内存中的written为0，即让其可被shmwrite进程写入数据。

2、程序shmwrite取得共享内存并连接到自己的地址空间中。检查共享内存中的written，是否为0，若不是，表示共享内存中的数据还没有被完，则等待其他进程读取完成，并提示用户等待。若共享内存的written为0，表示没有其他进程对共享内存进行读取，则提示用户输入文本，并再次设置共享内存中的written为1，表示写完成，其他进程可对共享内存进行读操作。

## 关于前面的例子的安全性讨论

这个程序是不安全的，当有多个程序同时向共享内存中读写数据时，问题就会出现。可能你会认为，可以改变一下written的使用方式，例如，只有当written为0时进程才可以向共享内存写入数据，而当一个进程只有在written不为0时才能对其进行读取，同时把written进行加1操作，读取完后进行减1操作。这就有点像文件锁中的读写锁的功能。咋看之下，它似乎能行得通。**但是这都不是原子操作，所以这种做法是行不能的**。试想当written为0时，如果有两个进程同时访问共享内存，它们就会发现written为0，于是两个进程都对其进行写操作，显然不行。当written为1时，有两个进程同时对共享内存进行读操作时也是如此，当这两个进程都读取完是，written就变成了-1.

**要想让程序安全地执行，就要有一种进程同步的进制，保证在进入临界区的操作是原子操作**。例如，可以**使用前面所讲的信号量来进行进程的同步。因为信号量的操作都是原子性的**。

五、使用共享内存的优缺点

1、优点：我们可以看到使用共享内存进行进程间的通信真的是非常方便，而且函数的接口也简单，**数据的共享还使进程间的数据不用传送，而是直接访问内存，也加快了程序的效率**。同时，它也**不像匿名管道那样要求通信的进程有一定的父子关系**。

2、缺点：**共享内存没有提供同步的机制**，这使得我们在使用共享内存进行进程间通信时，往往要借助其他的手段来进行进程间的同步工作。





















































































































