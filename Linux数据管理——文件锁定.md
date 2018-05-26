# Linux数据管理——文件锁定

## 什么是文件锁定

而因为程序经常需要共享数据，而这通常又是通过文件来实现的，试想一个情况，A进程正在对一个文件进行写操作，而另一个程序B需要对同一个文件进行读操作，并以读取到的数据作为自己程序运行时所需要的数据，这会发生什么情况呢？进程B可能会读到错乱的数据，因为它并不知道另一个进程A正在改写这个文件中的数据。

为了解决类似的问题，就出现了文件锁定，简单点来说，这是文件的一种安全的更新方式，当一个程序正在对文件进行写操作时，文件就会进入一种暂时状态，在这个状态下，如果另一个程序尝试读这个文件，它就会自动停下来等待这个状态结束。Linux系统提供了很多特性来实现文件锁定，其中最简单的方法就是以原子操作的方式创建锁文件。

用回之前的例子就是，文件锁就是当文件在写的时候，阻止其他的需要写或者要读文件的进程来操作这个文件。

## 创建锁文件

创建一个锁文件是非常简单的，我们可以使用open系统调用来创建一个锁文件，在调用open时oflags参数要增加参数O_CREAT和O_EXCL标志，如：

```c++
file_desc = open("/tmp/LCK.test", O_RDWR|O_CREAT|O_EXCL, 0444);
```

就可以创建一个锁文件/tmp/LCK.test。O_CREAT|O_EXCL，可以确保调用者可以创建出文件，**使用这个模式可以防止两个程序同时创建同一个文件**，如果文件（/tmp/LCK.test）已经存在，则open调用就会失败，返回-1。

**如果一个程序在它执行时，只需要独占某个资源一段很短的时间，这个时间段（或代码区）通常被叫做临界区，我们需要在进入临界区之前使用open系统调用创建锁文件，然后在退出临界区时用unlink系统调用删除这个锁文件。**

**注意：锁文件只是充当一个指示器的角色，程序间需要通过相互协作来使用它们，也就是说锁文件只是建议锁，而不是强制锁，并不会真正阻止你读写文件中的数据**。

可以看看下面的例子：源文件文件名为filelock1.c，代码如下：

```c++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>

int main()
{
	const char *lock_file = "/tmp/LCK.test1";
	int n_fd = -1;
	int n_tries = 10;

	while(n_tries--)
	{
                //创建锁文件
		n_fd = open(lock_file, O_RDWR|O_CREAT|O_EXCL, 0444);
		if(n_fd == -1)
		{
                        //创建失败
			printf("%d - Lock already present\n", getpid());
			sleep(2);
		}
		else
		{
                        //创建成功
			printf("%d - I have exclusive access\n", getpid());
			sleep(1);
			close(n_fd);
                        //删除锁文件，释放锁
			unlink(lock_file);
			sleep(2);
		}
	}
	return 0;
}
```

同时运行同一个程序的两个实例，运行结果为：

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86%E2%80%94%E2%80%94%E6%96%87%E4%BB%B6%E9%94%81%E5%AE%9A/1.jpg)\

从运行的结果可以看出两个程序交叉地对对文件进行锁定，但是真实的操作却是，每次调用open函数去检查/tmp/LCK.test1这个文件是否存在，如果存在open调用就失败，显示有进程已经把这个文件锁定了，如果这个文件不存在，就创建这个文件，并显示许可信息。但是这种做法有一定的缺憾，我们可以看到文件/tmp/LCK.test1被创建了很多次，也被unlink删除了很多次，也就是说我们不能使用已经事先有数据的文件作为这种锁文件，因为如果文件已经存在，则open调用总是失败。

给我的感觉是，这更像是一种对进程工作的协调性安排，更像是二进制信号量的作用，文件存在为0，不存在为1，而不是真正的文件锁定。

## 区域锁定

我们还有一个问题，就是如果同一个文件有多个进程需要对它进行读写，而一个文件同一时间只能被一个进程进行写操作，但是多个进程读写的区域互不相关，如果总是要等一个进程写完其他的进程才能对其进行读写，效率又太低，那么是否可以让多个进程同时对文件进行读写以提高数据读写的效率呢？

为了解决上面提到的问题，和出现在第二点中的问题，即**不要把文件锁定到指定的已存在的数据文件上**的问题，我们提出了一种新的解决方案，就是**区域锁定**。

简单点来说，区域锁定就是，文件中的某个部分被锁定了，但其他程序可以访问这个文件中的其他部分。

然而，区域锁定的创建和使用都比上面说的文件锁定复杂很多。

### 创建区域锁定

在Linux上为实现这一功能，我们可以使用fcntl系统调用和lockf调用，但是下面以fcntl系统调用来讲解区域锁定的创建。

fctnl的函数原理为：

```c++
int fctnl(int fildes, int command, ...);
```

它**对一个打开的文件描述进行操作**，并能根据command参数的设置完成不同的任务，它有三个可选的任务：F_GETLK，F_SETLK,F_SETLKW，至于这三个参数的意义下面再详述。而当使用这些命令时，fcntl的第三个参数必须是一个指向flock结构的指针，所以在实际应用中，fctnl的函数原型一般为：

```c++
int fctnl(int fildes, int command, struct flock *flock_st);
```

### flock结构

准确来说，flock结构依赖具体的实现，但是它至少包括下面的成员：

short l_type;文件锁的类型，对应于F_RDLCK（读锁，也叫共享锁），F_UNLCK（解锁，也叫清除锁），F_WRLCK（写锁，也叫独占锁）中的一个。

short l_whence;从文件的哪个相对位置开始计算，对应于SEEK_SET（文件头），SEEK_CUR（当前位置），SEEK_END(文件尾）中的一个。

off_t l_start;从l_whence开始的第l_start个字节开始计算。

off_t l_len;锁定的区域的长度。

pid_t l_pid;用来记录持有锁的进程。

成员l_whence、l_start和l_len定义了一个文件中的一个区域，即一个连续的字节集合，例如：

struct flock region;

region.l_whence = SEEK_SET;

region.l_start = 10;

region.l_len = 20;

则表示fcntl函数操作锁定的区域为文件头开始的第10到29个字节之间的这20个字节。

### 文件锁的类型

从上面的flock的成员l_type的取值我们可以知道，文件锁的类型主要有三种，这里对他们进行详细的解说。

F_RDLCK：

从它的名字我们就可以知道，它是一个**读锁，也叫共享锁**。**许多不同的进程可以拥有文件同一（或重叠）区域上的读（共享）锁**。而且**只要任一进程拥有一把读（共享）锁，那么就没有进程可以再获得该区域上的写（独占）锁**。**为了获得一把共享锁，文件必须以“读”或“读/写”方式打开**。

简单点来说就是，当一个进程在读文件中的数据时，文件中的数据不能被改变或改写，这是为了防止数据被改变而使读数据的程序读取到错乱的数据，而文件中的同一个区域能被多个进程同时读取，这是容易理解的，因为读不会破坏数据，或者说读操作不会改变文件的数据。

F_WRLCK：

从它的名字，我们就可以知道，它是一个**写锁，也叫独占锁**。**只有一个进程可以在文件中的任一特定区域拥有一把写（独占）锁**。**一旦一个进程拥有了一把写锁，任何其他进程都无法在该区域上获得任何类型的锁**。为了获得一把写（独占）锁，文件也必须以“读”或“读/写”方式打开。

简单点来说，就是一个文件同一区域（或重叠）在同一时间，只能有一个进程能对其进行写操作，并且在写操作期间，其他的进程不能对该区域读取数据。这个要求是显然易见的，因为如果两个进程同时对一个文件进行写操作，就会使文件的内容错乱起来，而由于写操作会改变文件中的数据，所以它也不允许其他进程对文件的数据进行读取和删除文件等操作。

F_UNLCK:

从它的名字就可以知道，它用于把一个锁定的区域解锁。

### 不同的command的意义

在前面说到fcntl函数的command参数时，说了三个命令选项，这里将对它们进行详细的解说。

F_GETLK命令，它用于获取fildes（fcntl的第一个参数）打开的文件的锁信息，它不会尝试去锁定文件，调用进程可以把自己想创建的锁类型信息传递给fcntl，函数调用就会返回将会阻止获取锁的任何信息，即它可以测试你想创建的锁是否能成功被创建。fcntl调用成功时，返回非-1，如果锁请求可以成功执行，flock结构将保持不变，如果锁请求被阻止，fcntl会用相关的信息覆盖flock结构。失败时返回-1。

所以，如果调用成功，调用程序则可以通过检查flock结构的内容来判断其是否被修改过，来检查锁请求能否被成功执行，而又因为l_pid的值会被设置成拥有锁的进程的标识符，所以大多数情况下，可以通过检查这个字段是否发生变化来判断flock结构是否被修改过。

使用F_GETLK的fcntl函数调用后会立即返回。

举个例子来说，例如，有一个flock结构的变量，flock_st,flock_st.l_pid = -1，文件的第10~29个字节已经存在一个读锁，文件的第40~49个字节中已经存在一个写锁，则调用fcntl时，如果用F_GETLK命令，来测试在第10~29个字节中是否可以创建一个读锁，因为这个锁可以被创建，所以，fcntl返回非-1，同时，flock结构的内容也不会改变，flock_st.l_pid = -1。而如果我们测试第40~49个字节中是否可以创建一个写锁时，由于这个区域已经存在一个写锁，测试失败，但是fcntl还是会返回非-1，只是flock结构会被这个区域相关的锁的信息覆盖了，flock_st.l_pid为拥有这个写锁的进程的进程标识符。

F_SETLK命令，这个命令试图对fildes指向的文件的某个区域加锁或解锁，它的功能根据flock结构的l_type的值而定。而对于这个命令来说，flock结构的l_pid字段是没有意义的。如果加锁成功，返回非-1，如果失败，则返回-1。使用F_SETLK的fcntl函数调用后会立即返回。

F_SETLKW命令，这个命令与前面的F_SETLK，命令作用相同，但不同的是，它在无法获取锁时，即测试不能加锁时，会一直等待直到可以被加锁为止。

### 例子

看了这么多的说明，可能你已经很乱了，就用下面的例子来整清你的思想吧。

源文件名为filelock2.c，用于创建数据文件，并将文件区域加锁，代码如下：

```c++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>

int main()
{
	const char *test_file = "test_lock.txt";
	int file_desc = -1;
	int byte_count = 0;
	char *byte_to_write = "A";
	struct flock region_1;
	struct flock region_2;
	int res = 0;

	//打开一个文件描述符	
	file_desc = open(test_file, O_RDWR|O_CREAT, 0666);
	if(!file_desc)
	{
		fprintf(stderr, "Unable to open %s for read/write\n", test_file);
		exit(EXIT_FAILURE);
	}
	//给文件添加100个‘A’字符的数据
	for(byte_count = 0; byte_count < 100; ++byte_count)
	{
		write(file_desc, byte_to_write, 1);
	}
	//在文件的第10～29字节设置读锁（共享锁）
	region_1.l_type = F_RDLCK;
	region_1.l_whence = SEEK_SET;
	region_1.l_start = 10;
	region_1.l_len = 20;
	//在文件的40～49字节设置写锁（独占锁）
	region_2.l_type = F_WRLCK;
	region_2.l_whence = SEEK_SET;
	region_2.l_start = 40;
	region_2.l_len = 10;

	printf("Process %d locking file\n", getpid());
	//锁定文件
	res = fcntl(file_desc, F_SETLK, ®ion_1);
	if(res == -1)
	{
		fprintf(stderr, "Failed to lock region 1\n");
	}
	res = fcntl(file_desc, F_SETLK, ®ion_2);
	if(res == -1)
	{
		fprintf(stderr, "Failed to lock region 2\n");
	}
	//让程序休眠一分钟，用于测试
	sleep(60);
	printf("Process %d closing file\n", getpid());
	close(file_desc);
	exit(EXIT_SUCCESS);
}
```

下面的源文件filelock3.c用于测试上一个文件设置的锁，测试可否对两个区域都加上一个读锁，代码如下：

```c++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>

int main()
{
	const char *test_file = "test_lock.txt";
	int file_desc = -1;
	int byte_count = 0;
	char *byte_to_write = "A";
	struct flock region_1;
	struct flock region_2;
	int res = 0;
	//打开数据文件
	file_desc = open(test_file, O_RDWR|O_CREAT, 0666);
	if(!file_desc)
	{
		fprintf(stderr, "Unable to open %s for read/write\n", test_file);
		exit(EXIT_FAILURE);
	}
	//设置区域1的锁类型
	struct flock region_test1;
	region_test1.l_type = F_RDLCK;
	region_test1.l_whence = SEEK_SET;
	region_test1.l_start = 10;
	region_test1.l_len = 20;
	region_test1.l_pid = -1;
	//设置区域2的锁类型
	struct flock region_test2;
	region_test2.l_type = F_RDLCK;
	region_test2.l_whence = SEEK_SET;
	region_test2.l_start = 40;
	region_test2.l_len = 10;
	region_test2.l_pid = -1;
	//对区域1的是否可以加一个读锁进行测试
	res = fcntl(file_desc, F_GETLK, ®ion_test1);
	if(res == -1)
	{
		fprintf(stderr, "Failed to get RDLCK\n");
	}
	if(region_test1.l_pid == -1)
	{
		//可以加一个读锁
		printf("test: Possess %d could lock\n", getpid());
	}
	else
	{
		//不允许加一个读锁
		printf("test:Prossess %d  get lock failure\n", getpid());
	}

	//对区域2是否可以加一个读锁进行测试
	res = fcntl(file_desc, F_GETLK, ®ion_test2);
	if(res == -1)
	{
		fprintf(stderr, "Failed to get RDLCK\n");
	}
	if(region_test2.l_pid == -1)
	{
		//可以加一个读锁
		printf("test: Possess %d could lock\n", getpid());
	}
	else
	{
		//不允许加一个读锁
		printf("test:Prossess %d  get lock failure\n", getpid());
	}
	exit(EXIT_SUCCESS);
}
```

运行结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86%E2%80%94%E2%80%94%E6%96%87%E4%BB%B6%E9%94%81%E5%AE%9A/2.jpg)

因为区域1中存在的是读锁，所以在其之上再加一个读锁是可以成功的，然而区域2上存在的锁是写锁，在其上不能加任何类型的锁，所以测试失败。注意，测试失败并不是fctnl调用失败，它还是返回非-1，我们是通过检查flock结构的成员l_pid来确定测试结果的。

### 解空锁问题

如果我要给在本进程中没有加锁的区域解锁会发生什么事情呢？而如果这个区域中其他的进程有对其进行加锁又会发生什么情况呢？

如果一个进程实际并未对一个区域进行锁定，而调用解锁操作也会成功，但是它并不能解其他的进程加在同一区域上的锁。也可以说解锁请求最终的结果取决于这个进程在文件中设置的任何锁，没有加锁，但对其进行解锁得到的还是没有加锁的状态。

























































































