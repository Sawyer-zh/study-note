# Linux进程间通信——使用信号量

## 什么是信号量

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的。

**信号量是一个特殊的变量，程序对其访问都是原子操作**，且只允许对它进行等待（即P(信号变量))和发送（即V(信号变量))信息操作。**最简单的信号量是只能取0和1的变量**，这也是信号量最常见的一种形式，叫做二进制信号量。而**可以取多个正整数的信号量被称为通用信号量**。这里主要讨论二进制信号量。

## 信号量的工作原理

由于信号量只能进行两种操作等待和发送信号，即P(sv)和V(sv)，他们的行为是这样的：

P(sv)：如果sv的值大于零，就给它减1；如果它的值为零，就挂起该进程的执行

V(sv)：如果有其他进程因等待sv而被挂起，就让它恢复运行，如果没有进程因等待sv而挂起，就给它加1.

举个例子，就是两个进程共享信号量sv，一旦其中一个进程执行了P(sv)操作，它将得到信号量，并可以进入临界区，使sv减1。而第二个进程将被阻止进入临界区，因为当它试图执行P(sv)时，sv为0，**它会被挂起以等待第一个进程离开临界区域并执行V(sv)释放信号量，这时第二个进程就可以恢复执行**。

## Linux的信号量机制

Linux提供了一组精心设计的信号量接口来对信号进行操作，它们不只是针对二进制信号量，下面将会对这些函数进行介绍，但请注意，这些函数都是用来对成组的信号量值进行操作的。它们声明在头文件sys/sem.h中。

### 1、semget函数

它的作用是创建一个新信号量或取得一个已有信号量，原型为：

```c++
int semget(key_t key, int num_sems, int sem_flags);
```

**第一个参数key是整数值（唯一非零），不相关的进程可以通过它访问一个信号量**，它代表程序可能要使用的某个资源，程序对所有信号量的访问都是间接的，程序先通过调用semget函数并提供一个键，再由系统生成一个相应的信号标识符（semget函数的返回值），**只有semget函数才直接使用信号量键，所有其他的信号量函数使用由semget函数返回的信号量标识符**。如果多个程序使用相同的key值，key将负责协调工作。

第二个参数num_sems**指定需要的信号量数目**，它的值**几乎总是1**。

第三个参数sem_flags是一组标志，当想要当信号量不存在时创建一个新的信号量，可以和值IPC_CREAT做按位或操作。**设置了IPC_CREAT标志后，即使给出的键是一个已有信号量的键，也不会产生错误**。而**IPC_CREAT | IPC_EXCL则可以创建一个新的，唯一的信号量，如果信号量已存在，返回一个错误**。

semget函数成功返回一个相应信号标识符（非零），失败返回-1。

### 2、semop函数

它的作用是改变信号量的值（也就是根据sem_op设置的值，来进行+1或者-1操作），原型为：

```c++
int semop(int sem_id, struct sembuf *sem_opa, size_t num_sem_ops); // 返回值是+1或-1之后的信号量的值
```

sem_id是由semget返回的信号量标识符，sembuf结构的定义如下：

```c++
struct sembuf{  
    short sem_num;//除非使用一组信号量，否则它为0  
    short sem_op;//信号量在一次操作中需要改变的数据，通常是两个数，一个是-1，即P（等待）操作，  
                    //一个是+1，即V（发送信号）操作。  
    short sem_flg;//通常为SEM_UNDO,使操作系统跟踪信号量，  
                    //并在进程没有释放该信号量而终止时，操作系统释放信号量  
};
```

### 3、semctl函数

该函数用来直接控制信号量信息，它的原型为：

```c++
int semctl(int sem_id, int sem_num, int command, ...);
```

如果有第四个参数，它通常是一个union semum结构，定义如下：

```c++
union semun{  
    int val;  
    struct semid_ds *buf;  
    unsigned short *arry;  
};
```

前两个参数与前面一个函数中的一样，command通常是下面两个值中的其中一个。

SETVAL：**用来把信号量初始化为一个已知的值**。**这个值通过union semun中的val成员设置，其作用是在信号量第一次使用前对它进行设置**。

IPC_RMID：用于删除一个已经无需继续使用的信号量标识符。

## 进程使用信号量通信的代码例子

下面使用一个例子来说明进程间如何使用信号量来进行通信，这个例子是两个相同的程序同时向屏幕输出数据，我们可以看到如何使用信号量来使两个进程协调工作，使同一时间只有一个进程可以向屏幕输出数据。注意，如果程序是第一次被调用（为了区分，第一次调用程序时带一个要输出到屏幕中的字符作为一个参数），则需要调用set_semvalue函数初始化信号并将message字符设置为传递给程序的参数的第一个字符，同时第一个启动的进程还负责信号量的删除工作。**如果不删除信号量，它将继续在系统中存在，即使程序已经退出，它可能在你下次运行此程序时引发问题，而且信号量是一种有限的资源**。

在main函数中调用semget来创建一个信号量，该函数将返回一个信号量标识符，保存于全局变量sem_id中，然后以后的函数就使用这个标识符来访问信号量。

源文件为seml.c，代码如下：

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/sem.h>

union semun
{
	int val;
	struct semid_ds *buf;
	unsigned short *arry;
};

static int sem_id = 0;

static int set_semvalue();
static void del_semvalue();
static int semaphore_p();
static int semaphore_v();

int main(int argc, char *argv[])
{
	char message = 'X';
	int i = 0;

	//创建信号量
	sem_id = semget((key_t)1234, 1, 0666 | IPC_CREAT);

	if(argc > 1)
	{
		//程序第一次被调用，初始化信号量
		if(!set_semvalue())
		{
			fprintf(stderr, "Failed to initialize semaphore\n");
			exit(EXIT_FAILURE);
		}
		//设置要输出到屏幕中的信息，即其参数的第一个字符
		message = argv[1][0];
		sleep(2);
	}
	for(i = 0; i < 10; ++i)
	{
		//进入临界区
		if(!semaphore_p())
			exit(EXIT_FAILURE);
		//向屏幕中输出数据
		printf("%c", message);
		//清理缓冲区，然后休眠随机时间
		fflush(stdout);
		sleep(rand() % 3);
		//离开临界区前再一次向屏幕输出数据
		printf("%c", message);
		fflush(stdout);
		//离开临界区，休眠随机时间后继续循环
		if(!semaphore_v())
			exit(EXIT_FAILURE);
		sleep(rand() % 2);
	}

	sleep(10);
	printf("\n%d - finished\n", getpid());

	if(argc > 1)
	{
		//如果程序是第一次被调用，则在退出前删除信号量
		sleep(3);
		del_semvalue();
	}
	exit(EXIT_SUCCESS);
}

static int set_semvalue()
{
	//用于初始化信号量，在使用信号量前必须这样做
	union semun sem_union;

	sem_union.val = 1;
	if(semctl(sem_id, 0, SETVAL, sem_union) == -1)
		return 0;
	return 1;
}

static void del_semvalue()
{
	//删除信号量
	union semun sem_union;

	if(semctl(sem_id, 0, IPC_RMID, sem_union) == -1) // 把第三个参数设为IPC_RMID即可删除信号量
		fprintf(stderr, "Failed to delete semaphore\n");
}

static int semaphore_p()
{
	//对信号量做减1操作，即等待P（sv）
	struct sembuf sem_b;
	sem_b.sem_num = 0;
	sem_b.sem_op = -1;//P()
	sem_b.sem_flg = SEM_UNDO;
	if(semop(sem_id, &sem_b, 1) == -1) // 如果减1之后，信号量变成了-1
	{
		fprintf(stderr, "semaphore_p failed\n");
		return 0;
	}
	return 1;
}

static int semaphore_v()
{
	//这是一个释放操作，它使信号量变为可用，即发送信号V（sv）
	struct sembuf sem_b;
	sem_b.sem_num = 0;
	sem_b.sem_op = 1;//V()
	sem_b.sem_flg = SEM_UNDO;
	if(semop(sem_id, &sem_b, 1) == -1)
	{
		fprintf(stderr, "semaphore_v failed\n");
		return 0;
	}
	return 1;
}
```

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%E2%80%94%E2%80%94%E4%BD%BF%E7%94%A8%E4%BF%A1%E5%8F%B7%E9%87%8F/1.jpg)

这个程序的临界区为main函数for循环中的semaphore_p和semaphore_v函数中间的代码。

例子分析 ：同时运行一个程序的两个实例，注意第一次运行时，要加上一个字符作为参数，例如本例中的字符‘O’，它用于区分是否为第一次调用，同时这个字符输出到屏幕中。因为每个程序都在其进入临界区后和离开临界区前打印一个字符，所以每个字符都应该成对出现，正如你看到的上图的输出那样。在main函数中循环中我们可以看到，每次进程要访问stdout（标准输出），即要输出字符时，每次都要检查信号量是否可用（即stdout有没有正在被其他进程使用）。所以，当一个进程A在调用函数semaphore_p进入了临界区，输出字符后，调用sleep时，另一个进程B可能想访问stdout，但是信号量的P请求操作失败，只能挂起自己的执行，当进程A调用函数semaphore_v离开了临界区，进程B马上被恢复执行。然后进程A和进程B就这样一直循环了10次。

## 对比例子——进程间的资源竞争

看了上面的例子，你可能还不是很明白，不过没关系，下面我就以另一个例子来说明一下，它实现的功能与前面的例子一样，运行方式也一样，都是两个相同的进程，同时向stdout中输出字符，只是没有使用信号量，两个进程在互相竞争stdout。它的代码非常简单，文件名为normalprint.c，代码如下：

```c++
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	char message = 'X';
	int i = 0;	
	if(argc > 1)
		message = argv[1][0];
	for(i = 0; i < 10; ++i)
	{
		printf("%c", message);
		fflush(stdout);
		sleep(rand() % 3);
		printf("%c", message);
		fflush(stdout);
		sleep(rand() % 2);
	}
	sleep(10);
	printf("\n%d - finished\n", getpid());
	exit(EXIT_SUCCESS);
}
```

例子分析：

从上面的输出结果，我们可以看到字符‘X’和‘O’并不像前面的例子那样，总是成对出现，因为当第一个进程A输出了字符后，调用sleep休眠时，另一个进程B立即输出并休眠，而进程A醒来时，再继续执行输出，同样的进程B也是如此。所以输出的字符就是不成对的出现。这两个进程在竞争stdout这一共同的资源。通过两个例子的对比，我想信号量的意义和使用应该比较清楚了。

## 信号量的总结

**信号量是一个特殊的变量，程序对其访问都是原子操作**，且只允许对它进行等待（即P(信号变量))和发送（即V(信号变量))信息操作。我们通常通过信号来解决多个进程对同一资源的访问竞争的问题，使在任一时刻只能有一个执行线程访问代码的临界区域，也可以说它是协调进程间的对同一资源的访问权，也就是用于同步进程的。

















































































































































