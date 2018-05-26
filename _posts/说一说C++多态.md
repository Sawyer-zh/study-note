---
title: 说一说C++多态
date: 2017-11-03 18:56:57
tags:
- C++
---

何为多态？我们简单的说一下，**在面向对象语言中，接口的多种不同的实现方式即为多态**。在C++中，多态分为**静态多态**和**动态多态**。怎么理解这两种多态呢？下面，我们通过实践，来理解一下。

## 静态多态

静态多态是通过函数的重载来实现的。

首先，我给出一段代码：

```c++
#include <iostream>
using namespace std;

char foo(double d) {
    cout << "I am the first foo()" << endl;
}

char foo(int a)
{
    cout << "I am the second foo()" << endl;
}

int main(int argc, char const *argv[]) {
	foo(666.666);
    foo(666);

	return 0;
}
```

执行一下：

```shell
g++ test.cpp
./a
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/1.png)

OK，在这里，我们可以看到，我们调用了两次函数。这两次调用的函数都是同一个名字的，只是参数的类型有所不同（第一个是浮点型的，第二个是整型的）。但是，却调用了不同的函数。可以说，这操作还是挺骚的。

好的，现在让我们来看看为什么可以实现这种骚操作。

怎么看呢？我们来看看生成的那个可执行文件里面有什么蹊跷吧：

```shell
cat a.exe
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/2.png)

为什么看不了呢？可能因为`a.exe`是二进制文件的缘故吧，`cat`命令看不了，会乱码。

OK，我们换一种方式，使用`nm`命令看看：

```shell
nm a.exe
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/3.png)

nice，可以看了，我们接下来去锁定一下那两个同名的函数，使用`grep`命令：

```shell
nm a.exe | grep foo
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/4.png)

看到了没，在符号表中（图片中的那个T表示位于代码区的符号），那两个`foo`的表示形式是有所不同的，前面的`_Z3foo`是一样的，但是后面的一个字母是不同的，一个是`d`（表示的含义是double），一个是`i`（表示的含义是int）。也就是说，实际上我们的这两个函数是可以这样写的：

```c++
#include <iostream>
using namespace std;

char _Z3food(double d) {
    cout << "I am the first foo()" << endl;
}

char _Z3fooi(int a)
{
    cout << "I am the second foo()" << endl;
}

int main(int argc, char const *argv[]) {
	_Z3food(666.666);
    _Z3fooi(666);

	return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/5.PNG)

奶思，效果是一样的。

细心的小伙伴们可能发现了，返回值的类型`char`并没有在`_Z3food`和`_Z3fooi`这两个符号中体现出来。所以，这就是为什么**重载**只能是通过函数的形参列表的不同加以区分而不能通过返回值的类型加以区分的原因了。

综上所述，静态多态是在编译阶段就确定了要执行哪个函数。

## 动态多态

动态多态是通过虚函数来实现的。

首先，我给出一段代码：

```c++
#include <iostream>
using namespace std;

class Base {
public:
    virtual void sleep() {
        cout << "Base sleep" << endl;
    }
    virtual void eat() {
        cout << "Base eat" << endl;
    }
    virtual void run() {
        cout << "Base run" << endl;
    }
};

class Animal : public Base {
public:
    size_t age;
    
    void sleep() {
        cout << "Animal sleep" << endl;
    }
    void eat() {
        cout << "Animal eat" << endl;
    }
    void run() {
        cout << "Animal run" << endl;
    }
};

int main(int argc, char const *argv[]) {
	
    Animal dargon;
    Animal dog;
    
    Base* pBase = &dargon;
    Base& pRe = dog;
    
    //通过基类的指针指向派生类对象来实现动态多态
    pBase->sleep();
    //通过基类的引用指向派生类对象来实现动态多态
    pRe.sleep();
    
	return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/6.PNG)

又是骚操作，指针类型是基类的，执行的却是子类的函数。

好的，现在让我们来看看为什么可以实现这种骚操作。

我们这次从汇编的代码来看看为什么可以有这样的骚操作：

```shell
g++ -S test.cpp
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/7.png)

我们现在就针对这个汇编代码来研究一番。

### 虚函数表

首先，我们来先看一看文件中一个叫做`_ZTV4Base`的东西：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/8.png)

这是什么呢？我们可以通过`c++filt`命令来转换：

```shell
c++filt _ZTV4Base
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/9.PNG)

翻译过来就是**Base的虚函数表**。

OK，那这个虚函数表里面有什么东西呢？从图片中我们可以看到有：

```assembly
.quad   0
.quad   _ZTI4Base
.quad   _ZN4Base5sleepEv
.quad   _ZN4Base3eatEv
.quad   _ZN4Base3runEv
.globl  _ZTS6Animal
.section        .rdata$_ZTS6Animal,"dr"
.linkonce same_size
```

其中，它的第一项是0，第二项_ZIT4Base是关于Base的类型信息，这与typeid有关。我们不讨论它们。现在让我们来看看后面的几项。和刚才的方法一样，我们使用`c++filt`命令来分别查看它们：

```shell
c++filt _ZN4Base5sleepEv
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/10.PNG)

```shell
c++filt _ZN4Base3eatEv
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/11.PNG)

```shell
c++filt _ZN4Base3runEv
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/12.PNG)

可以看出，在这个虚函数表中，前三项刚好是按照类中定义顺序的那些三个虚函数。

我们继续看后面的：

```shell
c++filt _ZTS6Animal
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/13.PNG)

这个和_ZTI4Base那项一样是个数据符号，我们不讨论这一项。

类似的，我们来查看一下Animal的虚函数表：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/14.PNG)

OK，这张图片和上面的差不多，我们类推出在Animal虚函数表中，有`Animal::sleep`、`Animal::eat`、`Animal::run`这三个函数。

那么，还是没有解决“为什么指针类型是基类的，执行的却是子类的函数”这个问题。

### 类对象中，有指向虚函数表的指针

我们继续看汇编代码：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/15.PNG)

我们来看看，`_ZN4BaseC2Ev`是啥：

```shell
c++filt _ZN4BaseC2Ev
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/16.PNG)

发现它是`Base`类的构造函数。

我们来看看构造函数所做的工作是什么。我们在图片中可以看到这个：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/17.PNG)

这个不是上面出现的那个虚函数表吗？

实际上`16+$_ZTV4Base `就是Base类的虚函数表在内存中的地址。这也就是说，构造函数把虚表的地址给了一个变量，而这个变量，用来指向内存中Base类的虚函数表。因此，我们可以大胆的猜测一下，对应的C++代码是：

```c++
this->vtable = &Base_vtable;
```

类似的，我们来看看Animal类的构造函数做了什么：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/18.PNG)

我们在这张图片里面可以看到这个：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/19.PNG)

第一句是：`call    _ZN4BaseC2Ev`。了解汇编的同学一定知道这句话的意思是**子程序调用指令**。 也就是说，在执行Animal类的构造函数的时候，还执行了Base类的构造函数。

OK，继续看，后面的两句和Base类的构造函数所做的工作差不多，即把Animal类的虚函数表的地址给一个变量。因此，我们可以大胆的猜测一下，对应的C++代码是：

```c++
this->vtable = &Animal_vtable;
```

OK，也就是说，**因为虚函数的存在，使得类中多了一个指针变量来保存类的虚函数表的地址**。

我们来测试一下：

```c++
#include <iostream>
using namespace std;

class Base {
public:
    void sleep() {
        cout << "Base sleep" << endl;
    }
};

int main(int argc, char const *argv[]) {
    Base base;
    cout << sizeof(base) << endl;

    return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/30.PNG)

为什么这里会打印出1而不是保存sleep函数地址所需要的内存大小呢？

因为，函数的地址只与类型相关，而与类型的实例无关，编译器不会因为这个sleep函数而在实例内添加任何额外的信息。

那么为什么会打印出1呢？明明在类中没有定义任何变量呀。这是因为当我们声明该类型的实例的时候，它必须在内存中占有一定的空间，否则无法使用这些实例。（至于需要多少内存空间，由编译器决定。我这个编译器是分配1个字节的内存单元）

OK，现在，我们把sleep声明为虚函数：

```c++
#include <iostream>
using namespace std;

class Base {
public:
    virtual void sleep() {
        cout << "Base sleep" << endl;
    }
};

int main(int argc, char const *argv[]) {
    Base base;
    cout << sizeof(base) << endl;

    return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/31.PNG)

奶思，符合我们的预期。说明编译器为我们增加了一个指针变量。

虽然知道了这些，但是还是没有解决为什么执行的是子类的函数。

我们继续......

### 动态绑定

我们继续来看汇编代码。以下是主函数部分：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/20.PNG)

我们来回顾一下，进入主函数以后，执行的第一条C++语句：

```c++
Animal dargon;
```

对应的汇编代码是：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/21.PNG)

前面两句：

```assembly
leaq    -32(%rbp), %rax
movq    %rax, %rcx
```

其实是先移动栈指针，给变量dargon在栈上分配内存。变量dargon的首地址是 rbp - 32。

第三句是：

```assembly
call    _ZN6AnimalC1Ev
```

也就是执行了Animal类的构造函数。这个很容易理解，当使用静态方法：

```c++
classA object;
```

定义了一个类对象的时候，会调用构造函数。

OK，接下来看看C++的第二条语句：

```c++
Animal dog;
```

我们来看看对应的汇编代码：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/22.PNG)

和上面类似的，栈指针移动，为变量dog分配内存，然后，再调用Animal类的构造函数。变量dog的首地址是 rbp - 48。

OK，我们接下来看看C++的第三条语句：

```c++
Base* pBase = &dargon;
```

对应的汇编代码为：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/23.PNG)

因为我们什么分析了，变量dargon的首地址是 rbp - 32 ，所以，两句汇编的含义就是把：dargon变量的地址指针变量pBase。

我们继续看C++的第四条语句：

```c++
Base& pRe = dog;
```

对应的汇编代码为：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/24.PNG)

这句话和上面的类似。

我们继续看C++的第5和第6条语句：

```c++
pBase->sleep();
pRe.sleep();
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/29.PNG)

实现了动态多态的效果。

同理，后面的汇编代码也实现了动态多态：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/26.PNG)

### 再次测试

通过什么的分析，我们大致可以得出如下关于虚函数表的图：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/27.jpg)

#### 验证

```c++
#include <iostream>
using namespace std;

class Base {
public:
    virtual void sleep() {
        cout << "Base sleep" << endl;
    }
    virtual void eat() {
        cout << "Base eat" << endl;
    }
    virtual void run() {
        cout << "Base run" << endl;
    }
};

class Animal : public Base {
public:
    size_t age;
    
    void sleep() {
        cout << "Animal sleep" << endl;
    }
    void eat() {
        cout << "Animal eat" << endl;
    }
    void run() {
        cout << "Animal run" << endl;
    }
};

/*
    定义一个函数指针类型，类型为 void () (Animal * )；
    用于指向虚函数sleep，eat，run；
    这里之所以多出一个Animal * 参数是因为c++类的非静态成员函数，
    编译器会默认在参数列表开头加入指向类指针的参数
 */
typedef void (* pFun)(Animal * animal);

int main(int argc, char const *argv[]) {
	
    Animal dargon;
    Animal dog;
    
    Base* pBase = &dargon;
    Base& pRe = dog;
    
    //通过基类的指针指向派生类对象来实现动态多态
    pBase->sleep();
    //通过基类的引用指向派生类对象来实现动态多态
    pRe.sleep();

    /*
        取出Animal的虚表指针.
        (size_t *)&dargon => dargon起始地址转换为size_t *
        *(size_t *)&dargon => dargon起始地址开始取sizeof(size_t)个字节解析成size_t(虚表指针的值)
        (size_t *)*(size_t *)&dargon => 把这个值转换成size_t *类型
    
        size_t在32位机是4个字节，在64位机是8个字节，指针变量的大小和size_t的大小是一致的。
     */
    size_t* vptable_dargon = (size_t*)*(size_t*)&dargon;
    size_t* vptable_dog = (size_t*)*(size_t*)&dog;
    
    cout << "size_t size = " << sizeof(size_t) << endl;
    
    //一个类公用一个虚表指针
    if (vptable_dog == vptable_dargon)
    {
        cout << "vptable value is equal" << endl;
    }
    
    //遍历虚表指针
    while (*vptable_dargon)
    {
        //取出每个虚表函数
        pFun fun = (pFun)(*vptable_dargon);
        //调用每个虚表函数
        fun(&dargon);
        vptable_dargon++;
    }
    
	return 0;
}
```

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/28.PNG)

OK，和图片的猜想一致。

happy ending......

