---
title: 聊一聊PHP的对象
date: 2018-01-31 22:08:07
tags:
- PHP
---

今晚在过一遍PHP手册的时候，看了一下PHP的对象那一部分知识点。我发现还是有一些坑点的。

官网上有一个例子是这样的：

```php
<?php
class SimpleClass
{
    // property declaration
    public $var = 'a default value';

    // method declaration
    public function displayVar() {
        echo $this->var;
    }
}

$instance = new SimpleClass();

$assigned   =  $instance;
$reference  = &$instance;

$instance->var = '$assigned will have this value';

$instance = null; // $instance and $reference become null

var_dump($instance);
var_dump($assigned);
var_dump($reference);
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%81%8A%E4%B8%80%E8%81%8APHP%E7%9A%84%E5%AF%B9%E8%B1%A1/3.PNG)

通过代码的执行结果，我们可以看到，因为这一句：

```php
$instance = null;
```

而导致了变量`$instance`和`$reference`都为null。emmm，只看这一个结果还是很好解释的。因为

```php
$reference  = &$instance;
```

所以我们说`$reference`是`$instance`的引用。那么什么是引用呢？emmm，我先说一说什么是拷贝吧，这样会好理解一些。emmm，我从C语言的角度来说（因为PHP有一个叫做写时拷贝的机制，所以用PHP来举例子会麻烦些）。

有如下代码段：

```c
int a = 1;
int b = a;
```

那么，当执行这段代码的时候，在内存中会发生什么呢？

首先是执行`int a = 1;`，操作系统会为变量a分配4个字节的存储单元（一般int型是4个字节，具体多少，可以用sizeof(int)查看），然后再把数据1存放到这4个字节的存储单元中。

可以简单的用如下图片表示（这里不考虑机器字节序和网络字节序，反正我是记不住，哈哈哈）：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%81%8A%E4%B8%80%E8%81%8APHP%E7%9A%84%E5%AF%B9%E8%B1%A1/4.PNG)

然后再执行语句`int b = a;`。









