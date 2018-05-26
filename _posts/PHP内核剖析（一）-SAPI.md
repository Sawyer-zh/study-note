---
title: PHP内核剖析(一)--SAPI
date: 2017-12-19 18:06:09
tags:
- PHP
---

额，菜鸡博主以后要把PHP作为主要开发语言。emmm，所以正在阅读PHP的源码。博主最近在阅读秦朋先生的《PHP内核剖析》，所以呢，这篇博客属于总结性的。

对了，前几天有一家公司的技术官就问了我对于Fpm的理解，当时我只是大概的知道它是用来管理进程的，却不是很清楚它具体做了些什么。因为我前期偏向于做更底层一些的事情，也就是在传输层和网络层，在应用层上面花的时间少了一些。以至于我对PHP本身的原理不是很清楚。这主要是受韩天峰老师当时对我的一些建议性指导的影响，让我先把基础打牢，然后再来研究那些框架呀等等，这样取得的成就更高。现在看来确实如此，如果没有去学习那些Linux知识、网络编程、并发编程的知识，要想理解那些高级的知识简直难于登天，真的非常感谢！

OK，进入正题。这次与大家分享的是PHP的SAPI（Server API），顾名思义就是我要完成一个PHP服务所需要的接口。在这里我只说其中的两个：Cli、Fpm。

## Cli

Cli（Command Line Interface）是命令行接口。也就是说，我们可以通过命令行的方式来执行PHP脚本。这个SAPI我记得是我当初在用PHP写爬虫的时候第一次使用的。我那个时候刚学PHP（或者说刚学编程）不久，然后看了一本PHP写网页爬虫的书籍，拿着书上的代码，通过Web服务器在浏览器里面跑这个爬虫脚本。跑着跑着，发现电脑的风扇在快速的转动......然后我才想起，在浏览器里面跑循环电脑不是很OK。所以寻找了另一种方案，于是找到了通过Cli的方式来跑爬虫脚本。这是我初始Cli。

OK，现在我来稍微详细的介绍一下PHP的Cli。

首先，在Cli模式执行PHP程序，它定义了很多命令行参数，不同的参数对应不同的处理，例如：执行PHP脚本、直接执行PHP代码（-r）、输出PHP版本（-v）等等。

### 执行流程

Cli是单进程模式，处理完请求后就直接关闭了，生命周期先后经历了`module startup`、`request startup`、`execute script`、`request shutdown`、`module shutdown`，关键过程如下：

```
main() -> php_cli_startup -> do_cli() -> php_module_shutdown()
```

Cli SAPI的主函数位于`/sapi/cli/php_cli.c`，当我们通过形如`php -r "echo 1;"`的方式执行一段PHP代码的时候，会先解析这个`-r`参数，然后再来初始化`sapi_module_struct`。这个结构是用来记录SAPI信息的（具体有哪些信息可以看PHP源码）。

完成了`sapi_module_struct`的初始化之后，就进入了`module startup`阶段。它是通过`sapi_module_struct`结构中的定义的一个`php_cli_startup`函数来进入这个阶段的。

在完成了`module startup`阶段后，就进入请求初始化阶段（request startup阶段）。在这个阶段，有一个关键的函数`do_cli`，它将完成请求的处理。

完成了请求初始化阶段之后，就可以开始执行PHP代码了（也就是我们上面的`echo 1;`）。

当`do_cli()`完成以后，回到主函数，进入`module shutdown`阶段。

emmm，这就是Cli模式下，进程所经历的生命周期。

## Fpm

Fpm就比Cli要复杂许多了。Fpm（FastCGI Process Manager）是一个进程管理器。FastCGI 是Web服务器和处理程序之间的一种通信协议，它是与HTTP类似的一种应用层通信协议（它只是一种协议），定义好了自己的输入和输出。

在网络应用场景下，PHP并没有像Golang那样实现HTTP网络库，而是实现了FastCGI协议，然后与Web服务器配合实现了HTTP的处理，Web服务器来处理HTTP请求，然后将解析的结果再通过FastCGI协议转发给处理程序（PHP脚本），处理程序完成后将结果返回给Web服务器，Web服务器再返回给用户。如图：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%86%85%E6%A0%B8%E5%89%96%E6%9E%90%28%E4%B8%80%29--SAPI/1.PNG)

PHP实现了FastCGI协议的处理，但是并没有实现具体的网络处理，比较采用的网络处理模型有以下几种。

- 多进程模型（Nginx就是采用这种模型）
- 多线程模型（memcached就是这种模型）

### 基本实现

Fpm是一种多进程模型，它由一个master进程和多个worker组成。master进程启动时会创建一个socket，但是不会接收、处理请求，而是由fork出的子进程完成请求的接收及处理。

master进程的主要工作是管理worker进程，负责fork或杀掉worker进程。根据worker进程的空闲状态来决定是fork还是杀掉worker进程。

worker进程的主要工作是处理请求，每个worker进程会竞争地Accept请求，接收成功后解析FastCGI，然后执行相应的脚本，处理完成后关闭请求，继续等待新的请求，这就是一个worker进程的生命周期。从worker进程的生命周期可以看到：一个worker进程只能处理一个请求，只有将一个请求处理完后才会处理下一个请求。这与Nginx的事件模型有很大的区别，Nginx的子进程epoll管理套接字，它是非阻塞的模型，只处理活跃的套接字。Fpm的这种处理模式大大简化了PHP的资源管理，使得在Fpm模式下不需要考虑并发导致的资源冲突（但是某些地方还是存在竞争的，例如父进程master获得子进程worker的信息是通过共享内存的方式来获取的）。master进程与worker进程之间不会直接进行通信，master通过共享内存获取worker进程的信息（对应fpm_scoreboard_s结构），比如worker进程当前的状态。

Fpm可以同时监听多个端口，每个端口对应一个worker pool（对应fpm_worker_pool_s结构），每个pool下对应多个worker进程。每个worker pool之间构成一个链表。

如图：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%86%85%E6%A0%B8%E5%89%96%E6%9E%90%28%E4%B8%80%29--SAPI/2.PNG)

### Fpm初始化

首先进行SAPI的模块初始化，在这个阶段，会进行一系列的初始化操作。

#### fpm_conf_init_main()

解析`php-fpm.conf`配置文件，为每个worker pool分配一个fpm_worker_pool_s结构。

#### fpm_scoreboard_init_main()

分配用于记录worker进程运行信息的结构，此结构分配在共享内存上。并且，每个pool中的worker进程都对应了一个`fpm_scoreboard_proc_s`结构，用来记录这个pool中此该进程的信息。

#### fpm_signals_init_main()

这一步会创建一个管道，这个管道并不是用于master与worker进程通信的，它只是在master进程中使用。

#### fpm_sockets_init_main()

创建每个worker pool的socket套接字，启动后worker将监听此socket接收请求。

#### fpm_event_init_main()

启动master的事件管理，Fpm实现了一个事件管理器用于管理I/O、定时事件。

完成了上述的初始化操作之后，就是fpm_run操作了。此环节将fork出子进程，启动进程管理器，执行后master进程将不会返回这个函数，只有各worker进程会返回，也就是说main()函数中调用fpm_run()之后的操作均是worker进程的。

### worker--请求处理

worker进程不断Accept请求，有请求到达后将读取并解析FastCGI协议的数据，解析完成后开始执行PHP脚本，执行完成后关闭请求，继续监听等待新的请求到达。

- **等待请求：**worker进程阻塞的等待请求。
- **解析请求：**fastcgi请求到达后被worker接收，然后开始接收并解析请求数据，直到request数据完全到达。
- **请求初始化**
- **执行PHP脚本：**由`php_execute_script()`完成PHP脚本的编译、执行操作。
- **关闭请求**







