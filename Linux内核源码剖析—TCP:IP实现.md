# Linux内核源码剖析—TCP/IP实现

## 1、预备知识

### 内核空间与用户空间的接口

#### procfs

procfs 是一个虚拟文件系统，通常挂接在/proc目录下。

**网络代码注册的文件一般位于/proc/net录下**，在该目录下存在不少文件，有一种比较**特殊的文件**，**比如 tcp、udp**等。

**大多数网络功能在初始化时，都会在/proc/net中注册 一个或多个这样的文件**。当用户读取某个文件时，内核会调用一组内核函数来输出相应的信息。

#### sysctl(/proc/sys 目录)

sysctl接口运行用户读取或者修改内核参数。

procps 包里的 **sysctl 命令**可以用于配置 sysctl 接口导出的内核参数，实际上**这个命令是通过 /prco/sys目录下的文件与内核通信的**。

#### netlink套接口

netlink 套接口是网络应用程序与内核通信最新、最常用的接口。

netlink 套接口使用起来非常简单，通过套接口标准的 API 来打开、关闭，或者发送和接收信息。要在用户态创建一个 netlink 套接口，代码如下所示：

```c
socket(PF_ NETLINK, SOCK DGRAM, NETLINK ROUTE)
```

### 网络I/O加速

尽管技术有了巨大的进步，但是TCP/IP协议栈的处理方式却几乎没有变化。也就是说，**即便用户使用最先进的CPU，依然要处理那些未经优化的TCP/IP协议**，由此产生巨大的系统开销。例如，**TCP/IP的传输过程中需要封装、解包，这些动作对于处理器而言并不是一个复杂的过程，但是会占用处理器周期，而且网络带宽越高，这个问题越严重**。系统开销的增大不仅仅表现在占用较多的处理器周期，还会**导致处理网络相关数据时的内存访问效率降低**。这又会进一步降低 CPU 效能和网络效率。

过去，网络流量较低，处理网络相关数据所产生的开销，远远低于用于执行正常任务的开销，所以并未引起重视。现在，随着网络流量大幅度提升，处理网络相关数据所产生的系统开销越来越不能忽视，甚至已经影响到了正常应用。现有几种解决方案：

- TSO

  TSO 是通过网络设备进行 TCP 段的分割，从而来提高网络性能的一一种技术。较大的数据包（超过标准 1518 B 的帧）可以使用该技术，使操作系统减少必须处理的数据数量以提高性能。通常，当请求大量数据时，TCP 发送方必须将数据拆分为 MSS 大小的数据块，然后进一步将其封装为数据包形式，以便最终可以在网络中进行传输。而当启用了 TSO 技术之后，TCP 发送方可以将数据拆分为 MSS 整数倍大小的数据块，然后将大块数据的分段直接交给网络设备处理，操作系统需要创建并传输的数据包数量更少，因此性能会有较大的提高。图 1-3 所示为标准帧和 TSO 技术特性比较。

  ![](http://oklbfi1yj.bkt.clouddn.com/Linux%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E2%80%94TCP/IP%E5%AE%9E%E7%8E%B0/1.png)

- RDMA

- Onloading

### slab分配器

Linux 所使用的 slab 分配器的基础是 Jeff Bonwick 为 SunOS 操作系统首次引入的一种算法。与传统的内存管理模式相比，slab 缓存分配器有很多优点。首先，内核通常依赖于对小对象的分配，它们会在系统生命周期内进行无数次分配。slab 缓存分配器通过对类似大小的对象进行缓存而提供这种功能，从而**避免了常见的碎片问题**。**slab 分配器还支持通用对象的初始化，从而避免了为同一目的而对一个对象重复进行初始化**。最后，**slab 分配器还可以支持硬件缓存对齐和着色，这允许不同缓存中的对象占用相同的缓存行，从而提高缓存的利用率并获得更好的性能**。

### RCU

RCU (Read-Copy Update），顾名思义就是读-拷贝更新，它是基于其原理命名的。**对于被 RCU 保护的共享数据结构，“读者”不需要获得任何锁就可以访问它，但“写者”在访问它时将首先拷贝一个副本，然后对副本进行修改，最后使用一个回调机制在适当的时机把指向原来数据的指针重新指向新的被修改的数据**。**这个时机就是所有引用该数据的 CPU 都退出对共享数据的操作**。

RCU 实际上是一种改进的 rwlock，“读者”不需要锁，不使用原子指令，几乎没有什么同步开销，而且在除 alpha 的所有架构上也不需要内存栅（Memory Barrier），因此不会导致锁竞争。而“写者”的同步开销比较大，它需要延迟数据结构的释放，复制被修改的数据结构。它还必须使用某种锁机制同步并行的其他“写者”的修改操作。“读者”必须提供一个信号给“写者”以便“写者”能够确定数据可以被安全地释放或修改的时机。有一个专门的垃圾收集器来探测“读者”的信号，一旦所有的“读者”都已经发送信号告知它们都未使用被 RCU 保护的数据结构，垃圾收集器就调用回调函数完成最后的数据释放或修改操作。RCU 与 rwlock 的不同之处是：它既允许多个“读者”同时访问被保护的数据，又允许多个“读者”和多个“写者”同时访问被保护的数据，“读者”没有任何同步开销，而“写者”的同步开销则取决于使用的“写者”间同步机制。

ps：读写锁比mutex有更高的适用性，可以多个线程同时占用读模式的读写锁，但是只能一个线程占用写模式的读写锁。

1. 当读写锁是**写加锁状态**时，在这个锁被解锁之前，所有试图对这个锁加锁的线程都会被阻塞；
2. 当读写锁在**读加锁状态**时，所有试图以读模式对它进行加锁的线程都可以得到访问权，但是以写模式对它进行枷锁的线程将阻塞；
3. 当读写锁在**读模式锁状态**时，如果有另外线程试图以**写模式加锁**，**读写锁通常会阻塞随后的读模式锁请求**，这样可以避免读模式锁长期占用，而等待的写模式锁请求长期阻塞。

这种锁**适用对数据结构进行读的次数比写的次数多的情况**下，**因为可以进行读锁共享**。

## 2、网络体系结构概述

### 网络架构

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E2%80%94TCP/IP%E5%AE%9E%E7%8E%B0/2.png)

应用层通常是一个语义层，能够理解要传输的数据。

下图在最上面是用户空间中实现的应用层，而**中间为内核空间中实现的网络子系统**，底部为物理设备，提供了对网络的连接能力。其中中间部分正是本书的重点所在，**在网络协议栈（指的是下图中间的部分）内部流动的是套接口缓冲区（SKB），用于在协议栈的底层、上层以及应用层之间传递报文数据**。

网络协议栈顶部是**系统调用接口，为用户空间中的应用程序提供一种访问内核网络子系统的接口**。下面是一个协议无关层（即套接口层），它提供了一种通用方法来使用传输层协议。然后是传输层的具体协议，包括 TCP、UDP。在传输层下面是网络层。然后是邻居子系统，**在邻居子系统存在的目标才是当前可以直接访问的**。再下面是网络设备接口，提供了与各个设备驱动程序通信的通用接口。最底层是设备驱动程序。

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E2%80%94TCP/IP%E5%AE%9E%E7%8E%B0/3.png)

### 系统调用接口

网络子系统提供了**两种调用接口**给用户进程。

用户进程在进行网络调用时，通过系统特有的网络调用接口进入内核。在内核中，进一步调用`sys_ socketcall()`结束该过程，在`sys_ socketcall()`中会根据网络系统调用号调用具体的功能。

**另一种系统调用接口是通过普通文件操作来访问网络子系统**。虽然有很多操作是网络专用的，比如使用 socket 系统调用创建一个套接口，使用 connect 系统调用连接一个服务器等，但**套接口的输入/输出操作可以被当成典型的文件读写操作来进行**。

### 协议无关层

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E2%80%94TCP/IP%E5%AE%9E%E7%8E%B0/4.png)

通过网络协议栈通信都需要对套接口进行操作。套接口层是一个与协议无关的接口，它提供了一组接口来支持各种协议。套接口层不但可以支持典型的 TCP 和 UDP 协议，还可以支持 RAW 套接口、RAW 以太网和其他传输协议，如SCTP。

**Linux 中用 socket 结构描述套接口，代表一条通信链路的一端，用来存储与该链路有关的所有信息**。这些信息包括所使用的**协议、协议的状态信息（包括源和目的地址）、到达的连接队列、数据缓存和可选标志**等。其中**最关键的成员是 sk 和 ops，前者指向与该套接口相关的传输控制块，后者指向特定传输协议的操作集**，见图 2-3。

### 传输层协议

如图 2-4 所示，套接口的 sk 字段指向与该套接口相关的传输控制块，传输层使用传输控制块存放套接口所需的信息。传输控制块按协议而异，TCP 传输控制块、UDP 传输控制块、RAW 传输控制块，分别对应 tcp_ sock 结构（支持完整的 TCP 特性，包含了 TCP 为各连接维护的所有结点信息：两个方向的序号、窗口大小、重传次数等），udp_ sock 结构（支持完整的 UDP 特性，包括两端 IP 地址、两端端口号、本端 IP 选项等）和 raw sock 结构。

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E2%80%94TCP/IP%E5%AE%9E%E7%8E%B0/5.png)











































































































