# list

一个特点就是对于指针，如果这个指针的值不会变化的话，那么都使用了`const`。

## 整个list的结构

有一些和编译器有关的东西不是很清楚？例如`portBASE_TYPE`

![](http://oklbfi1yj.bkt.clouddn.com/%E6%88%91%E5%AF%B9FreeRTOS%E7%9A%84%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/list/1.png)

## 初始化list

初始化后的列表如图所示：

![](http://oklbfi1yj.bkt.clouddn.com/%E6%88%91%E5%AF%B9FreeRTOS%E7%9A%84%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/list/2.png)

uxNumberOfItems被初始化为0；

xListEnd.xItemValue初始化为0xffffffff（每个列表数据结构体中都有一个列表项成员xListEnd，用于标记列表结束。xListEnd.xItemValue被初始化为一个常数，其值与硬件架构相关，为0xFFFF或者0xFFFFFFFF。这个常数在移植层定义，即宏portMAX_DELAY）；

pxIndex、xListEnd.pxNext和xListEnd.pxPrevious初始化为指向列表项xListEnd。

## 插入list item

![](http://oklbfi1yj.bkt.clouddn.com/%E6%88%91%E5%AF%B9FreeRTOS%E7%9A%84%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/list/3.png)

如果在添加一个list item（假设它的值是22），那么是加在32和xListEnd之间。

如果在添加一个list item（假设它的值是34），那么是加在32的后面。

## 删除list item

这个被删除的list item没有被delete掉？

