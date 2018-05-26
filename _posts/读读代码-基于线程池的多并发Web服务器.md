---
title: 读读代码-基于线程池的多并发Web服务器
date: 2017-12-02 10:10:38
tags:
- C语言
- 并发
- 线程池
- 网络编程
---

这次与大家分享的是我阅读的一个**基于线程池**的多并发Web服务器，收获很多，非常感谢[这位兄弟的代码](https://github.com/bjut-hz/HttpServer)。

如果大家不是很清楚什么是线程池，可以看[百度百科的解释](https://baike.baidu.com/item/%E7%BA%BF%E7%A8%8B%E6%B1%A0/4745661?fr=aladdin)。简单的来说，线程池有如下特点：

- 线程数量
- 工作线程
- 任务接口
- 任务队列

这些特点会在后面的代码中体现出来（其实也是设计线程池的一些基本内容）。

那么我们为什么需要使用线程池呢？

如果客户端发起一个请求，服务器就创建一个线程来处理该请求，处理完了之后，销毁该线程，那么开销是相当大的。在实际使用中，为每个请求创建新线程的服务器在创建和销毁线程上花费的时间和消耗的系统资源，甚至可能要比花在处理实际的用户请求的时间和资源要多得多（在遇到这种**大量短小请求**时，适合使用线程池）。除了创建和销毁线程的开销之外，活动的线程也需要消耗系统资源。因此，我们可以使用这种**池化技术**来减少资源不足，尽量利用已有对象来进行服务。 （这种思路和内存池有些像，也是使用池化技术）

OK，介绍完了线程池和它的优点之后，小伙伴们应该对它有了一些了解。接下来就是枯燥的阅读代码时刻（我也觉得很枯燥，但是想想可以学到新知识，就会坚持读下去，这个过程需要反复的去查找资料）。

这次的代码量有些大了，超过1000行了（但是无论多么的长，我们只要找到主函数，就可以一直读下去。注意**使用一个好的IDE或者文本编辑器**，这些东西可以大大的提高我们的阅读效率），所以呢，我只挑我个人认为需要分享的地方咯......

<!-- more -->

首先，我们来看一看线程池的结构（在`threadpool.c`文件中）：

```c++
/**
 * 线程池的结构定义
 *  @var lock         用于内部工作的互斥锁
 *  @var notify       线程间通知的条件变量
 *  @var threads      线程数组，这里用指针来表示，数组名 = 首元素指针（数组是指针的语法糖）
 *  @var thread_count 线程数量
 *  @var queue        存储任务的数组，即任务队列
 *  @var queue_size   任务队列大小
 *  @var head         任务队列中首个任务位置（注：任务队列中所有任务都是未开始运行的）
 *  @var tail         任务队列中最后一个任务的下一个位置（注：队列以数组存储，head 和 tail 指示队列位置）
 *  @var count        任务队列里的任务数量，即等待运行的任务数
 *  @var shutdown     表示线程池是否关闭（不太理解）
 *  @var started      开始的线程数
 */
struct threadpool_t {
  pthread_mutex_t lock;
  pthread_cond_t notify;
  pthread_t *threads;
  threadpool_task_t *queue;
  int thread_count;
  int queue_size;
  int head;
  int tail;
  int count;
  int shutdown;
  int started;
};
```

上面的注释是代码作者之前已经写好了的。我这里就讲讲这些变量的作用吧。

- lock

  顾名思义，锁的意思。它的作用是用来使线程同步的，为了防止竞争，条件变量的使用总是和一个互斥锁结合在一起。（有一个需要注意的地方就是，这个互斥锁不是用来保护条件变量的，因为条件变量是原子操作。这个互斥锁是用来保护wait的）

- notify

  条件变量也是一种同步机制，主要包括两个动作：一个线程等待条件变量的条件成立，另一个线程使条件成立（在这里，等待条件变量的条件成立的线程是线程池中的那些线程；条件变量的条件是任务队列中是否有任务；使条件成立的线程是main主线程，它负责往任务队列中添加任务）

OK，我们现在从主函数开始读起（因为与socket相关的代码部分，我们之前讲过了，所以我们这里主要是讲与线程池有关的部分）。

首先是创建一个线程池（在`httpserver.c`文件中）：

```c++
pool = threadpool_create( THREAD, QUEUE, 0 )
```

这个函数是创建一个线程池，线程池中线程的最大个数是THREAD个，任务队列中任务的最大个数是QUEUE个。

我们继续看看`threadpool_create()`函数内部定义：

```c++
pool = (threadpool_t *)malloc(sizeof(threadpool_t))
```

这条语句是申请一块内存，用来存放线程池。

分配了内存给线程池用之后，需要给该线程池进行初始化（这一点和new操作符不一样，new操作符会调用构造函数，在构造函数中初始化线程池中的变量）：

```c++
pool->threads = (pthread_t *)malloc(sizeof(pthread_t) * thread_count);
```

这条语句是申请一块内存，用来存放线程。

```c++
/* 创建指定数量的线程开始运行 */
for(i = 0; i < thread_count; i++) {
    if(pthread_create(&(pool->threads[i]), NULL,threadpool_thread, (void*)pool) != 0){
        threadpool_destroy(pool, 0);
        return NULL;
  }
  
    pool->thread_count++;
    pool->started++;
}
```

这段代码是创建指定数量的线程。

接着我们来看看`pthread_create`函数中每个线程在跑的`threadpool_thread`函数做了些什么。

```c++
    for(;;) {
        /* Lock must be taken to wait on conditional variable 锁必需等待wait*/
        /* 取得互斥锁资源 */
        pthread_mutex_lock(&(pool->lock));

        /* Wait on condition variable, check for spurious wakeups. 等待条件变量，检查虚假唤醒
           When returning from pthread_cond_wait(), we own the lock. 从pthread_cond_wait（）返回时，我们拥有锁。*/
        /* 用 while 是为了在唤醒时重新检查条件 */
        while((pool->count == 0) && (!pool->shutdown)) {
            /* 任务队列为空，且线程池没有关闭时阻塞在这里 */
            pthread_cond_wait(&(pool->notify), &(pool->lock));
        }

        /* 关闭的处理 */
        if((pool->shutdown == immediate_shutdown) ||
           ((pool->shutdown == graceful_shutdown) &&
            (pool->count == 0))) {
            break;
        }

        /* Grab our task */
        /* 取得任务队列的第一个任务 */
        task.function = pool->queue[pool->head].function;
        task.argument = pool->queue[pool->head].argument;
        /* 更新 head 和 count */
        pool->head += 1;
        pool->head = (pool->head == pool->queue_size) ? 0 : pool->head; // 相当于实现一个循环队列。其实也可以使用取模来实现循环的效果。
        pool->count -= 1;

        /* Unlock */
        /* 释放互斥锁 */
        pthread_mutex_unlock(&(pool->lock));

        /* Get to work */
        /* 开始运行任务 */
        (*(task.function))(task.argument);
        /* 这里一个任务运行结束 */
    }
```

注意，因为刚开始初始化线程池的时候，任务队列里面是没有任务的，所以开始创建的线程都会因为条件变量的条件不满足而阻塞（此时的阻塞状态不是一直占用CPU等待，而是休眠起来。之后main主线程会往任务队列里面添加任务，此时线程池中的线程会被唤醒。所以初始化线程池的操作就是为了**让线程们休眠起来**）。

可能有小伙伴对：

```c++
(*(task.function))(task.argument);
```

会有一些疑问，我们在主线程main中的后面会有讲解。

好的，我们继续返回到main主线程里面读代码，在`while`循环中，main主线程去接收客户端的请求，然后往线程池中的任务队列里面添加任务：

```c++
threadpool_add( pool, &accept_request, (void*)&client_sock, 0 )
```

我们来看看`threadpool_add`函数，首先来看看函数声明的部分：

```c++
int threadpool_add(threadpool_t *pool, void (*function)(void *),
                   void *argument, int flags)
```

这个函数的第一个参数是一个线程池对象。第二个参数是一个函数指针。可以看出，这是在C语言里面实现**回调函数**的一个技巧。而第三个参数就是回调函数的一个参数。第四个参数没有用上。

继续看：

```c++
if(pthread_mutex_lock(&(pool->lock)) != 0) {
    return threadpool_lock_failure;
}
```

刚开始的时候，我一直在想，为什么这里需要获得互斥锁？不是只有主线程main在往线程池中的任务队列里面添加任务吗？没有其他的线程在往任务队列中添加任务了呀？

然后我发现了这个模型实际上是一个**生存者与消费者的模型**。我在学习操作系统的时候，就遇到了这个问题。加锁是为了解决生存者与消费者之间的同步问题。在这里，生存者是main主线程，而消费者是线程池里面的线程。加锁是为了防止生存者在生产（添加任务）的时候，消费者过来了消费（获取任务）。

继续看代码：

```c++
pthread_cond_signal(&(pool->notify))
```

`pthread_cond_signal`只能唤醒已经处于pthread_cond_wait的线程。也就是说，如果signal的时候没有线程在condition wait，那么本次signal就没有效果，**后续的线程进入condition wait之后，无法被之前的signal唤醒**。

OK，现在我们就可以解释：

```c++
(*(task.function))(task.argument);
```

是什么意思了。这个实际上就是处理客户端请求的一个函数（任务）。参数是一个表示连接的socket文件描述符。

额，既然我们学习了一个基于线程池的Web服务器，那么，我们应该就想要知道它处理多个请求的能力如何？于是，我上Github找了一个工具，叫做：[Webbench](https://github.com/EZLippi/WebBench)。具体的使用方法可以[参考这里](https://www.cnblogs.com/xxyBlogs/p/5639103.html)。（注意，在url结尾的处要使用`/`，例如：`./webbench -c 10 http://www.baidu.com/ `）

我测试的时候，使用几百个客户端对这个服务器（不是百度的这个，是我们读的这个服务器）发起请求还是比较稳定的，但是当我使用1000个客户端的时候，服务器就不稳定了，有时候会直接死掉。

好的，这次的分享就到这里了。总的来说这个小项目还是需要一定的基础的，例如是否可以看出添加任务和执行任务实际上就是我们所学的生产者与消费者模型；使用循环队列这个数据结构来存放任务（这个又和任务调度的知识点有关了）；操作系统这一块的知识就更多了，必须要懂什么是线程，如何做到线程安全，怎么解决消费者想要获取任务而任务队列中却没有任务的问题等等。

总之，有一个好的基础非常重要。（尽管我一直在打基础，但是还是有很多地方理解不到位，所以呀，必须得理论结合实践嘛）

OK，接着学习其他项目，待我去Github上面筛选一波......ps：其实我挺想读Swoole这个网络通信框架的，然而呵呵了。能力还是有限......以后一定读，毕竟我想要那个啥......

happy ending......