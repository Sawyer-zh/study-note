---
title: nginx数据结构容器--链表
date: 2018-01-28 22:08:18
tags:
- nginx
- 数据结构
---

今天继续阅读了陶辉前辈的《深入理解Nginx模块开发与架构解析》，看到了Nginx的数据结构部分。因为书中并没有提供详细的源码，所以，只看书不动手做实验总是感觉很不踏实。

于是我今晚把Nginx中链表和内存池中的代码抽取了出来，够折腾的，到处改代码。希望我的浪费时间可以给大家节约时间。本篇博客的代码 [在这里](https://github.com/huanghantao/ngxdatastructure/tree/master/ngx_list)。

首先，我从设计的角度给大家介绍下Nginx中的链表。首先我们要知道，Nginx对内存分配非常吝啬（因为只有保证低内存消耗，才有可能实现巨大的同时并发连接数），所以**链表是存放在内存池上面的**。

OK，知道这个结论之后，我给大家介绍一下Nginx中与链表有关的数据结构：

## nginx_list_t数据结构

```c
typedef struct ngx_list_part_s ngx_list_part_t;
struct ngx_list_part_s {
    void *elts;
    ngx_uint_t nelts;
    ngx_list_part_t *next;
}
```

```c
typedef struct {
    ngx_list_part_t *last;
    ngx_list_part_t part;
    size_t size;
    ngx_uint_t nalloc;
    ngx_pool_t *pool;
} ngx_list_t;
```

其中`ngx_list_t`结构体中的`ngx_pool_t *pool;`成员就是用来给`ngx_list_t`配置一个内存池用的。

OK，知道了与链表相关的数据结构之后，我来用一幅图表示一下它们之间的联系以便于小伙伴们对于后续代码的理解：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/1.PNG)

[图片来自这里，侵删](http://hankjin.blog.163.com/blog/static/33731937201091111303630/)

现在我们通过代码一步一步的来理解这个数据结构如何使用以及内存分布的样子。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ngx_palloc.h"
#include "ngx_list.h"

int main(int argc, char const *argv[])
{
    ngx_pool_t *ngx_mem_pool = ngx_create_pool(1024, NULL);
    ngx_list_t *testlist = ngx_list_create(ngx_mem_pool, 6, sizeof(int));

    if (NULL == testlist) {
    	exit('创建链表失败');
    }
    
    ngx_list_init(testlist, ngx_mem_pool, 6, sizeof(int));
    ngx_int_t *var = ngx_list_push(testlist);
    *var = 1;

    ngx_list_part_t *part = &(testlist->part);

    ngx_int_t *var1 = part->elts;
    
    int i;
    for (i = 0; /* void */; ++i) {
        if (i >= part->nelts) {
        	if (NULL == part->next) {
        		break;
        	}

        	part = part->next;
        	var1 = part->elts;
        	i = 0;
        }

        printf("list element:%d\n", var1[i]);
    }

    return 0;
}
```

我们执行一下这段代码：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/2.PNG)

OK，代码执行没问题，我来讲解一下代码。

```c
ngx_pool_t *ngx_mem_pool = ngx_create_pool(1024, NULL);
```

这段代码是用来创建一个内存池的，内存池的大小为1024字节。返回值是内存池的地址。

```c
ngx_list_t *testlist = ngx_list_create(ngx_mem_pool, 6, sizeof(int));

if (NULL == testlist) {
    exit('创建链表失败');
}

ngx_list_init(testlist, ngx_mem_pool, 6, sizeof(int));
```

`ngx_list_create`用来在内存池`ngx_mem_pool`上面创建新的链表。6指的是每个链表数组可容纳元素的个数是6（相当于ngx_list_t结构体中的nalloc成员）。`sizeof(int)`是每个元素的数据大小，这里我们想要在链表中存放整型数据。该函数的返回值是创建的链表的地址。

好的，我们来打印一下内存池的地址和链表的地址，看看它们的关系，代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ngx_palloc.h"
#include "ngx_list.h"

int main(int argc, char const *argv[])
{
    ngx_pool_t *ngx_mem_pool = ngx_create_pool(1024, NULL);
    ngx_list_t *testlist = ngx_list_create(ngx_mem_pool, 6, sizeof(int));

    if (NULL == testlist) {
    	exit('创建链表失败');
    }

    printf("内存池地址:%p\n链表地址:%p\n", ngx_mem_pool, testlist);

    return 0;
}
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/3.PNG)

首先，我们知道，我们给内存池开的大小有1M，而栈的大小也就几兆。所以，讲道理内存池在实现上面应该不会开在栈上面，所以是开在堆上面的。而堆的地址增长一般是从低地址到高地址的（详情查看C语言的内存模型），所以根据图中所示，链表首地址和内存池首地址只是相差了48个字节，远远小于1M。所以我们可以得出链表被分配在内存池上面的结论。我们运行代码再来测试一次：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/4.PNG)

依旧是相差48字节。咦( ′◔ ‸◔`)，这是很神奇的。为啥都是相差48字节呢？我们可以读读这些函数的实现来分析出答案。

OK，我们继续讲解第一份代码的剩余部分。

```c
 ngx_list_init(testlist, ngx_mem_pool, 6, sizeof(int));
```

是用来初始化一个已有的链表。我们可以看到，区别只是比`ngx_list_create`函数多了第一个参数。我们来看看调用了`ngx_list_create`和`ngx_list_init`之后，链表的区别在哪里，代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ngx_palloc.h"
#include "ngx_list.h"

int main(int argc, char const *argv[])
{
    ngx_pool_t *ngx_mem_pool = ngx_create_pool(1024, NULL);
    ngx_list_t *testlist = ngx_list_create(ngx_mem_pool, 6, sizeof(int));

    if (NULL == testlist) {
    	exit('创建链表失败');
    }
    
    printf("create函数调用之后的结果:\n");
    printf("%d\n", testlist->size);
    printf("%d\n", testlist->nalloc);
    printf("%p\n", testlist->pool);
    
    ngx_list_init(testlist, ngx_mem_pool, 6, sizeof(int));
    
    printf("init函数调用之后的结果:\n");
    printf("%d\n", testlist->size);
    printf("%d\n", testlist->nalloc);
    printf("%p\n", testlist->pool);

    return 0;
}
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/5.PNG)

可以看出，`ngx_list_t`这个结构体中的成员size、nalloc、pool在使用`ngx_list_create`函数的时候，已经进行了初始化。至于这两个函数的区别，我们还是需要从源码来分析。

OK，我们继续读代码的剩余部分。

```c
ngx_int_t *var = ngx_list_push(testlist);
*var = 1;
```

这段代码表示往链表`testlist`中添加一个元素，且返回值是这个元素的地址。我们可能会奇怪，为啥不直接`ngx_list_push(元素)`，而是要先`ngx_list_push(testlist)`后再来对其赋值呢？感觉有些脱了裤子来放屁啊！我开始读这个代码的时候也感到非常疑惑。但是细细一想，得到了答案：同一个内存池上面可能有其他链表，如果直接`ngx_list_push(元素)`，那么程序怎么知道要往哪个链表中添加元素呢？对吧。我们可以来测试一下，代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ngx_palloc.h"
#include "ngx_list.h"

int main(int argc, char const *argv[])
{
    ngx_pool_t *ngx_mem_pool = ngx_create_pool(1024, NULL);
    printf("内存池地址:%p\n", ngx_mem_pool);

    ngx_list_t *testlist_1 = ngx_list_create(ngx_mem_pool, 6, sizeof(int));
    printf("链表1的地址:%p\n", testlist_1);

    if (NULL == testlist_1) {
    	exit('创建链表失败');
    }
    
    ngx_list_init(testlist_1, ngx_mem_pool, 6, sizeof(int));
    ngx_int_t *var = ngx_list_push(testlist_1);
    *var = 1;


    ngx_list_t *testlist_2 = ngx_list_create(ngx_mem_pool, 6, sizeof(int));
    printf("链表2的地址:%p\n", testlist_2);

    if (NULL == testlist_2) {
        exit('创建链表失败');
    }
    
    ngx_list_init(testlist_2, ngx_mem_pool, 6, sizeof(int));
    var = ngx_list_push(testlist_2);
    *var = 2;

    ngx_list_part_t *part1 = &(testlist_1->part);

    ngx_int_t *var1 = part1->elts;
    
    int i;
    for (i = 0; /* void */; ++i) {
        if (i >= part1->nelts) {
        	if (NULL == part1->next) {
        		break;
        	}

        	part1 = part1->next;
        	var1 = part1->elts;
        	i = 0;
        }

        printf("list1 element:%d\n", var1[i]);
    }

    ngx_list_part_t *part2 = &(testlist_2->part);

    ngx_int_t *var2 = part2->elts;
    
    for (i = 0; /* void */; ++i) {
        if (i >= part2->nelts) {
            if (NULL == part2->next) {
                break;
            }

            part2 = part2->next;
            var2 = part2->elts;
            i = 0;
        }

        printf("list2 element:%d\n", var2[i]);
    }

    return 0;
}
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/6.PNG)

第一份代码的最后部分：

```c
int i;
for (i = 0; /* void */; ++i) {
    if (i >= part->nelts) {
        if (NULL == part->next) {
            break;
        }

        part = part->next;
        var1 = part->elts;
        i = 0;
    }

    printf("list element:%d\n", var1[i]);
}
```

这段代码的作用就是遍历出链表中的所有元素。

最后，再附上一张内存池上存放链表的图片：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AE%B9%E5%99%A8--%E9%93%BE%E8%A1%A8/7.png)

[图片来自这里，侵删](http://blog.csdn.net/u012377333/article/details/40582431)

好了，这篇博客到此结束了，希望小伙伴们有所收获。希望小伙伴们可以感受到通过我的博客，我在与你面对面交流，能够达到这个作用，我就很欣慰了。

happy ending......