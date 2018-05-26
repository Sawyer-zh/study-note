# Effective C++ 

## 1、尽量用 const 和 inline 而不用#define 

实际上它的意思是**尽量用编译器而不用预处理**。因为#define 经常被认为好象不是语言本身的一部分。例如：

```c++
#define ASPECT_RATIO 1.653
```

编译器会永远也看不到 ASPECT_RATIO 这个符号名，因为在源码进入编译器之前，它会被预处理程序去掉 ，于是 ASPECT_RATIO 不会加入到符号列表中。如果涉及到这个常量的代码在编译时报错，就会很令人费解，因为报错信息指的是 1.653，而不是 ASPECT_RATIO。

解决这个问题的方案很简单：不用预处理宏，定义一个常量：

```c++
const double ASPECT_RATIO = 1.653;
```

## 2、尽量用<iostream>而不用<stdio.h> 

scanf 和 printf 很轻巧，很高效，但是**它们不是类型安全**的，而且没有扩展性。

## 3、尽量用new和delete而不用malloc和free

千万别马虎地把 new 和 free 或 malloc 和 delete 混起来用，那只会自找麻烦。

## 4、内存管理

虚拟内存是个很好的发明但虚拟内存也是有限的，并不是每个人都可以最先抢到它。

### 对应的 new 和 delete 要采用相同的形式

即，调用 new 时用了[]，调用 delete 时也要用[]。如果调用 new 时没有用[]，那调用 delete 时也不要用[]。

```c++
string *stringarray = new string[100];
...
delete stringarray;
```

对于 delete 来说会有这样一个重要的问题：内存中有多少个对象要被删除？答案决定了将有多少个析构函数会被调用。
这个问题简单来说就是：要被删除的指针指向的是单个对象呢，还是对象数组？这只有你来告诉 delete。如果你在用 delete 时没用括号，delete 就会认为指向的是单个对象，否则，它就会认为指向的是一个数组。

在这里，stringarray 指向的 100 个 string 对象中的 99 个不会被正确地摧毁，因为他们的析构函数永远不会被调用。

用 new 的时候会发生两件事。首先，内存被分配(通过 operator new 函数)，然后，为被分配的内存调用一个或多个构造函数。用 delete 的时候，也有两件事发生：首先，为将被释放的内存调用一个或多个析构函数，然后，释放内存(通过 operator delete 函数)。

## 5、预先准备好内存不够的情况 

operator new 在无法完成内存分配请求时会抛出异常。

一个很简单的出错处理方法，可以这么做：当内存分配请求不能满足时，调用你预先指定的一个出错处理函数。这个方法基于一个常规，即当 operator new 不能满足请求时，会在抛出异常之前调用客户指定的一个出错处理函数——一般称为 new_handler 函数。 

指定出错处理函数时要用到 set_new_handler 函数，它在头文件`<new>`里大致是象下面这样定义的： 

```c++
typedef void (*new_handler)();
new_handler set_new_handler(new_handler p) throw();
```

可以看到，new_handler 是一个自定义的函数指针类型，它指向一个没有输入参数也没有返回值的函数 。
set_new_handler 则是一个输入并返回 new_handler 类型的函数。set_new_handler 的输入参数是 operator
new 分配内存失败时要调用的出错处理函数的指针，返回值是 set_new_handler 没调用之前就已经在起作用的旧
的出错处理函数的指针。 

可以象下面这样使用 set_new_handler：

```c++
// function to call if operator new can't allocate enough memory
void noMoreMemory() {
    cerr << "Unable to satisfy request for memory\n";
    abort();
}
int main() {
    set_new_handler(noMoreMemory);
    int *pBigDataArray = new int[100000000];
    ...
}
```

假如 operator new 不能为 100,000,000 个整数分配空间，noMoreMemory 将会被调用，程序发出一条
出错信息后终止。这就比简单地让系统内核产生错误信息来结束程序要好。 

```c++
assert(0); // assert 是个宏
```

## 6、写 operator new 和 operator delete 时要遵循常规 

自己重写 operator new 时，很重要的一点是函数提供的行为要和系统缺省的 operator new 一致。实际做起来也就是：要有正确的返回值；可用内存不够时要调用出错处理函数；处理好 0 字节内存请求的情况。此外，还要避免不小心隐藏了标准形式的 new。

C++标准要求，即使在请求分配 0 字节内存时，operator new 也要返回一个合法指针。(实际上，这个听起来怪怪的要求确实给 C++语言其它地方带来了简便) 这样，非类成员形式的 operator new 的伪代码看起来会象下面这样：

```c++
void * operator new(size_t size) // operator new 还可能有其它参数
{
  if (size == 0) { // 处理 0 字节请求时，
    size = 1; // 把它当作 1 个字节请求来处理
  }
  while (1) {
    分配 size 字节内存;
    if (分配成功)
      return (指向内存的指针);
    // 分配不成功，找出当前出错处理函数
    new_handler globalHandler = set_new_handler(0);
    set_new_handler(globalHandler);
    if (globalHandler) (*globalHandler)();
    else throw std::bad_alloc();
  }
}
```

处理零字节请求的技巧在于把它作为请求一个字节来处理。这看起来也很怪，但简单，合法，有效。而且，你又会多久遇到一次零字节请求的情况呢？

## 7、如果写了 operator new 就要同时写 operator delete 

为什么有必要写自己的 operator new 和 operator delete？

答案通常是：为了效率。缺省的 operator new 和 operator delete 具有非常好的通用性，它的这种灵活性也使得在某些特定的场合下，可以进一步改善它的性能。尤其在那些需要动态分配大量的但很小的对象的应用程序里情况更是如此。



























