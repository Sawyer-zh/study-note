# 从0到1写一个RTOS

注意，**每次调试之前，都需要先编译源代码**，否则调试的是上一次编译的结果。

## 1、课程总体介绍

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/1.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/2.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/3.png)

## 2、前后台代码结构

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/4.png)

### 缺点

#### 缺点1

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/5.png)

如上图所示，对事件2的处理必须延迟2秒后才能进行。

#### 缺点2

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/6.png)

如果resourceRdy这个资源一直没到来的话，那么cpu就会一直空转。为了防止cpu一直空转，我们设置了`delay > TIME_1S`这个条件，也就是说，当等待的时间超过了1s，就退出等待。（尽管这么做，但是还是浪费了cpu1s的时间）

如果我们把这一秒钟空出来，去做别的更有效的工作。然后当这个标志为为1的时候，我们再去执行事件处理的代码，这样cpu的利用率就会更高。但是，使用前后台这种代码结构是无法实现这样的操作的。而利用RTOS去做的话，这个过程可以很轻松的达到（也就是可以很轻松的使得cpu利用率提高）。

#### 缺点3

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/7.png)

#### 问题产生的原因

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/8.png)

## 3、RTOS原理及功能简介

### 概述

【摘自百度百科】实时操作系统（RTOS）是指当外界事件或数据产生时，能够接受并以足够快的速度予以处理，其处理的结果又能在规定的时间之内来控制生产过程或对处理系统做出快速响应，调度-一切可利用的资源完成实时任务，并控制所有实时任务协调一致运行的操作系统。提供及时响应和高可靠性是其主要特点。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/9.png)

实时并不意味着快，只需要**在规定的时间内完成即可**。

简单的说，RTOS 是一种通用的任务管理框架，用于控制任务的运行和任务之间的交互，保证事件得到实时处理。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/10.png)

### 工作原理

提供多个执行流：虽然实际只有一颗 CPU 硬件，但是通过“虚拟化”，每个 Task 好像独占 CPU。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/11.png)

可以看到，使用了RTOS之后，任务函数不需要返回值。但是对于前后台代码结构来说，任务函数就需要返回值，否则，一个任务一直执行，而其他的任务就得不到执行。

"虚拟”的 CPU 并非完全的虚拟，“独占”也并不是真正独占，而只是任务认为自己独占。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/12.png)

可以发现，使用了RTOS之后，通过任务的不断切换，有效利用了CPU空转的那部分时间。而使用前后台代码结构，则会浪费掉CPU空转的时间。

通过 RTOS 控制任务的运行时机，事件处理的实时性得到有效保证。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/13.png)

提供了一些组件用于简化任务对资源的访问，事件的处理，以及任务之间的通信，有效降低任务之间的代码耦合。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/14.png)

### 总结

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/15.png)

## 4、下载安装软件

MDK-ARM官方网站：`http://www.keil.com`

可以模拟硬件来进行调试我们的RTOS。

版本：5.21

## 5、创建初始工程

简单的示例：在 main（）中周期性对标志位进行周期性翻转。

```c
Void delay  (int count) {
    while  (--count> 0);
}

int flag;

int main  () {
    //可在逻辑分析仪中看到周期性变化的信号 for  (;;) {
    for (;;) {
        flag = 0;
    	delay (100);
    	flag = 1;
    	delay (100);
    }

	return 0;
}
```

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/16.png)

项目名称：

tinyRTOS

选择环境：

Devicd -> Startup

CMSS -> CORE

创建C文件：

Target 1 -> Source Group 1 -> Add New Item to Group Source Group 1

文件名为main.c

这个main.c文件放在tinyRTOS/Source下面

main函数里面一般是一个for循环。

## 6、调试工具使用

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/17.png)

## 6、芯片内核

### 概述

Cortex-M3 内核是 ARM 公司开发的 CPU 内核。完整的 MCU 芯片集成了 Cortex-M3 内核以及其它组件。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/18.png)

### 特性

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/19.png)

可以发现，这些特性和我们操作系统的很类似。实际上也是这样的，操作系统必须在硬件支持这些特性的情况下，操作系统才能够实现这些特性。

#### 工作模式及权限级别分类

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/20.png)

#### 内核寄存器组

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/21.png)

##### 程序状态寄存器

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/22.png)

其中，24位的那个标志位必须为1

#### 存储映射

##### Cortex-M3 预定义的存储器映射

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/23.png)

我们只需要了解如下区域：

SRAM：片内 RAM，用于存放堆栈、变量等数据

Code：用于存放可执行代码及常量

#### 堆栈

Cortex-M3 使用的是“向下生长的满栈”模型，采用双堆栈机制。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/24.png)

#### 异常/中断处理

##### 系统异常列表

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/25.png)

##### 进入异常

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/26.png)

寄存器压栈的顺序按照步骤一中图片的顺序进行。

##### 退出异常

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/27.png)

##### 复位异常响应序列

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/28.png)

芯片复位以后，默认会执行一个操作：

从我们的异常向量表里面加载PC和SP的值（这个异常向量表的起始地址一般是从0地址开始的）

得到PC和SP的值之后，赋值给对应的寄存器，如下图所示：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/29.png)

然后我们的CPU就开始运行了。

##### PendSV异常

这个异常是专门**用于RTOS上下文切换的**（即**不同任务之间的切换**）。工作原理：配置为最低优先级，上下文切换的请求将自动延迟到到其它的ISR都完成后才处理,并且可被其它异常/中断抢占。

配置为最低优先级，这样做的目的是：**保证所有中断都能得到实时、快速响应**

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/30.png)

#### 指令系统

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/31.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/32.png)

其中，`LDR RD, [Rs]`中的`[]`类似于C语言的`*`，即解引用，得到相应内存地址的数据（当然，这个数据可能还是一个地址）。

其中，`STMDB`中的D表示递减（即从高地址 -> 低地址），B表示before（即在之前）。举个例子：

```
STMDB R0!, {R4-R11}
```

含义：在写寄存器之前会先把R0的寄存器进行递减，然后再往里面写R4～R11。其中的`!`表示在里面R0地址中批量写了数据之后，最后那个被写入数据的单元的内存地址就保存在寄存器R0里面。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/33.png)

## 7、内核编程实践

### 需求说明

使用 PendSVC 触发异常，在异常处理函数中，保存 R4~R11 寄存器到缓冲区，再恢复 R4~R11 寄存器，以模拟任务切换时的寄存器保存与恢复。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/34.png)

其中，在`triggerPenSVC`这个函数里面，会有触发PendSVC异常的指令。

```c
#define NVIC_INT_CTRL       0xE000ED04      // 中断控制及状态寄存器
#define NVIC_PENDSVSET      0x10000000      // 触发软件中断的值
#define NVIC_SYSPRI2        0xE000ED22      // 系统优先级寄存器
#define NVIC_PENDSV_PRI     0x000000FF      // 配置优先级

void triggerPendSVC (void) 
{
    MEM8(NVIC_SYSPRI2) = NVIC_PENDSV_PRI;   // 向NVIC_SYSPRI2写NVIC_PENDSV_PRI，设置其为最低优先级
    MEM32(NVIC_INT_CTRL) = NVIC_PENDSVSET;    // 向NVIC_INT_CTRL写NVIC_PENDSVSET，用于PendSV
}
```

异常触发后，会对应执行`PendSV_Handler`（所以这个函数名字是固定的）：

```c

__asm void PendSV_Handler ()
{
    IMPORT  blockPtr
    
    // 加载寄存器存储地址
    LDR     R0, =blockPtr
    LDR     R0, [R0]
    LDR     R0, [R0]

    // 保存寄存器
    STMDB   R0!, {R4-R11}
    
    // 将最后的地址写入到blockPtr中
    LDR     R1, =blockPtr
    LDR     R1, [R1]
    STR     R0, [R1]
    
    // 修改部分寄存器，用于测试
    ADD R4, R4, #1
    ADD R5, R5, #1
    
    // 恢复寄存器
    LDMIA   R0!, {R4-R11}
    
    // 异常返回
    BX      LR
}  

```

这个异常处理函数是用来模拟任务之间的切换。我们知道，异常发生时，硬件会自动保存一些内核寄存器。例如sp、R0～R3。所以，剩下的工作就是我们自己需要去保存寄存器R4～R11。

下面这一部分就相当于去保存前一个任务的内核寄存器的值。

```assembly
LDR     R0, =blockPtr
LDR     R0, [R0]
LDR     R0, [R0]

// 保存寄存器
STMDB   R0!, {R4-R11}
```

下面的代码就是恢复下一个要运行任务的内核寄存器的值。

```assembly
// 恢复寄存器
LDMIA   R0!, {R4-R11}
```

下面这段代码就是异常返回，这样就返回到下一个任务去执行。这样就完成了任务的切换。

```assembly
// 异常返回
BX      LR
```

## 8、任务定义与切换原理

### 任务是什么

#### 任务的外观

一个永远不会返回的函数。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/35.png)

#### 任务的内在

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/36.png)

### 任务切换原理

任务切换的本质：保存前一任务的当前运行状态，恢复后一任务之前的运行状态。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/37.png)

#### 任务切换时候需要保存的状态

##### 任务状态数据

代码、数据区：由编译器自动分配，各个任务相互独立，并不冲突

堆：不使用

栈：硬件只支持两个堆栈空间（一个是用户栈，一个是内核栈），不同任务能否共用同一个栈？答案是不能的，因为当一个任务在使用栈的时候，如果RTOS去切换了一个任务来执行，那么这个新的任务如果执行了pop指令， 那么很可能会把前一个任务保存在栈中的数据弹出。

内核寄存器：编译器会在某些时间将值保存到栈中，如函数调用、异常处理。有一些内核寄存器硬件会自动帮我们保存，而其他的就需要我们自己去保存了。

其它的状态数据，如何处理？

#### 如何保存

##### 方法1

为每个任务配置独立的栈，用于保存该任务的所有状态数据。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/38.png)

这样，任务的切换过程就变成了：

 前一个任务的状态数据都保存在栈空间里面，然后选择下一个任务的堆栈，把栈空间之前保存的这些数据弹出来，这样就恢复了这个任务之前的运行状态，然后这个任务就继续往下运行了。

### 编码实现

#### 定义堆栈类型

```C
// Cortex-M的堆栈单元类型: 堆栈单元的大小为32位，所以使用uint32_t
// #include<stdint.h>，这个头文件里面有uint32_t这个类型的定义
typedef uint32_t tTaskStack;
```

#### 定义任务类型

```c
// 任务结构: 包含了一个任务的所有信息
typedef struct _tTask {
// 任务所用堆栈的当前堆栈指针。每个任务都有他自己的堆栈，用于在运行过程中存储临时变量等一些环境参数
// 在tiny0S运行该任务前，会从stack指向的位置处，会读取堆栈中的环境参数恢复到CPU寄存器中，然后开始运行
// 在切换至其它任务时，会将当前CPU寄存器值保存到堆栈中，等待下一次运行该任务时再恢复。
// stack保存了最后保存环境参数的地址位置，用于后续恢复
    uint32_t * stack;
}tTask;
```

#### 声明两个测试任务

```c
// 任务1和任务2的任务结构，以及用于堆栈空间]
tTask tTask1;
tTask tTask2;
tTaskStack task1Env[1024];
tTaskStack task2Env[1024];
```

## 9、任务切换的实现

### 设计目标

构建一个最小的两个任务切换运行的小系统，帮助了解RTOS任务切换的核心原理。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/39.png)

其中，图中的`tTaskSched`函数是一个调度函数，用来让当前任务主动释放cpu，然后另外一个任务占用cpu。

### 切换到初始任务

（注意，**每个任务都需要被初始化**）

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/40.png)

右边的栈是每个任务都有自己的栈空间。为了实现切换到初始任务，我们会先在初始任务的堆栈中预先的填充一些内核寄存器的初始值和状态数据的值。

当我们要切换到这个任务的时候，设置初始任务的栈为当前栈。然后将内核寄存器的值和状态数据的值从堆栈中弹出来，恢复到实际的硬件系统中。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/41.png)

在恢复内核寄存器的时候，会恢复R15的值，这个R15也就是PC指针的值。这个PC值在初始化任务的时候，会被初始化任务入口函数的地址。所以我们恢复了R15的值以后，CPU运行的就是初始任务函数的入口地址处。

### 任务之间的切换

切换过程（从任务1切换至任务2）

假设，在任务1运行的某个时间点上，任务1、任务2的堆栈、寄存器的状态如下：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/42.png)

接下来是切换的过程：

- 首先暂停运行任务1。这个暂停过程是在任务1中执行一个切换请求的操作，这个操作会触发PendSVC异常。在PendSVC异常处理函数中，我们会把内核寄存器的值以及状态数据保存在任务1的栈里面。然后我们再切换当前栈为任务2的栈。如下图所示：

  ![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/43.png)

- 从当前栈（任务2的栈）中恢复状态

  ![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/44.png)

  （如果任务2之前没有运行过的话，我们需要先初始化任务2的堆栈）

- 继续运行任务2的代码

### 设计实现

#### 任务初始化

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/45.png)

初始化任务为待恢复状态：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/46.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/47.png)

对上图的解释：

需要注意的一点就是，上面的顺序是有讲究的，不要改变。

因为状态寄存器的24位需要被置为1，因此是`1 << 24`。

entry是入口函数的地址，出栈的时候会被恢复给PC（也就是R15寄存器）。

param是任务的参数，这个值会被传递给R0寄存器。（为什么入口参数的值是被保存在R0寄存器呢？这个是由编译器来决定的，编译器默认的规则就是如果是一个函数，那么这个函数的第一个参数就是保存在R0寄存器里面）

#### 切换至初始任务

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/48.png)

解释一下上面的图片：

我们有两个任务，我们将第一个需要运行的任务赋值给`nextTask`，然后我们调用`tTaskRunFirst`函数来触发任务的切换，切换到第一个任务。注意，我们有一个设置PSP的值为0的操作。

切换的过程是通过设置PendSVC异常来完成的。

#### 从一个任务切换至后一个任务

我们实现了一个调度函数：调度函数的作用就是决定如何将CPU的控制权在不同任务之间进行分配。

因为我们只有两个任务，所以算法很简单，之间互换即可。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/49.png)

#### PendSVC异常处理函数实现

在这个异常处理函数里面，我们切换任务。

流程图如下：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/50.png)

异常发生的时候，会自动保存上图所示的一些寄存器。

PSP寄存器的作用：我们知道Cortex-M3芯片本身支持两种堆栈：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/51.png)

这两个堆栈分别使用不同的栈顶指针寄存器。在我们的RTOS操作系统中，所有的任务都是运行在用户级的状态中，所以堆栈都是用的PSP。正常情况下，PSP的值不应该为0。所以我们在切换至初始任务的时候，预先将PSP设置为了0，作为一个标记，说明了是切换到初始任务（也就是说这个任务是第一次被调度）。如果任务早就已经跑起来了的话，那么这个PSP当前肯定是指向某一个用户的栈里面的某个位置，这个值肯定是不等于0的。

现在的问题是，内核恢复以后，默认的是使用特权级堆栈，而我们希望的任务在运行的时候使用的是用户级的堆栈。那么如何设置？

方法如下：

我们可以在异常返回的时候，设置一下LR寄存器的值。这个LR寄存器在进入异常的时候会设置成下图的模式（也就是这些标识为）：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/52.png)

为了在退出异常的时候能够使用PSP，我们把第2位设置成了1。也就是说，出栈以后，我们使用用户级的堆栈，然后堆栈的指针就是PSP了。

**如果是两个任务之间进行切换，那么PSP的值是不等于0的**。这也是为什么初始化任务的时候，需要把PSP的值设置为0的原因。

### 思考

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/53.png)

## 10、双任务时间片运行原理

### 设计目标

构建一个最小的基于时间片切换的双任务小系统，帮助了解RTOS时间片任务切换的核心原理。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/54.png)

#### 涉及到的问题

##### 何时触发

让task1和task2各自占用一段cpu时间，它们的占用时间都是相等的。

##### 由谁来触发

既然是通过时间来划分，我们可以考虑使用一个定时器。

#### 运行效果

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/55.png)

#### 涉及要点

应该使用哪个定时器？定时器如何配置？

如何实现时间片的切换？

### 时间片实现

Cortex-M3内核自带了一个硬件定时器：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/56.png)

因为SysTick定时器是Cortex-M3内核标配的，所以不同的芯片都有这个定时器。

#### SysTick定时器

24 位的递减定时器，当递减到0时，将从RELOAD 寄存器中自动重载定时初值至CURRENT寄存器，如此反复。

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/57.png)

##### 配置SysTick定时器（例如，10ms）

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/58.png)

##### 定时中断切换任务

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/59.png)

##### 设置时钟频率

在Device/system_ARMCM3.c文件中：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/60.png)

### 效果

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AARTOS/61.gif)





















































































