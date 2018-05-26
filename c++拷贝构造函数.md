#  c++拷贝构造函数

首先对于普通类型的对象来说，它们之间的复制是很简单的，例如：

```c++
int a = 100;
int b = a;
```

而类对象与普通对象不同，类对象内部结构一般较为复杂，存在各种成员变量。

## 类对象拷贝的例子

是一种特殊的构造函数，**用基于同一类的一个对象构造和初始化另一个对象**。

```c++
#include<iostream>
using namespace std;

class CExample
{
private:
    int a;
public:
    //构造函数
    CExample(int b)
    {
        a=b;
        printf("constructor is called\n");
    }
    //拷贝构造函数
    CExample(const CExample & c)
    {
        a=c.a; // 可以看到，在类的方法中（类内）对象是可以直接对属性进行引用的
        printf("copy constructor is called\n");
    }
    //析构函数
    ~CExample()
    {
        cout<<"destructor is called\n";
    }
    void Show()
    {
        cout<<a<<endl;
    }
};

int main()
{
    CExample A(100);
    CExample B=A;
    B.Show(); 
    return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/1.png)

运行程序，屏幕输出100。从以上代码的运行结果可以看出，系统为对象 B 分配了内存并完成了与对象 A 的复制过程。就类对象而言，相同类型的类对象是通过拷贝构造函数来完成整个复制过程的。

```c++
CExample(const CExample& C)
```

就是我们自定义的拷贝构造函数。可见，拷贝构造函数是一种**特殊的构造函数**，函数的名称必须和类名称一致，它必须的一个参数是**本类型**的一个**引用变量**。

## 拷贝构造函数的调用时机

### 当函数的参数为类的对象时

```c++
#include<iostream>
using namespace std;

class CExample
{
private:
    int a;
public:
    CExample(int b)
    {
        a=b;
        printf("constructor is called\n");
    }
    CExample(const CExample & c)
    {
        a=c.a;
        printf("copy constructor is called\n");
    }
    ~CExample()
    {
     cout<<"destructor is called\n";
    }
    void Show()
    {
     cout<<a<<endl;
    }
};


void g_fun(CExample c)
{
    cout<<"g_func"<<endl;
}


int main()
{
    CExample A(100);
    CExample B=A;
    B.Show(); 
    g_fun(A);
    return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/2.png)

调用g_fun()时，会产生以下几个重要步骤：

(1).A对象传入形参时，会先会产生一个临时变量，就叫 C 吧。（所以会调用3次析构函数）

(2).然后**调用拷贝构造函数把A的值给C**。 整个这两个步骤有点像：CExample C(A);

(3).等g_fun()执行完后, 析构掉 C 对象。

### 函数的返回值是类的对象

```c++
#include<iostream>
using namespace std;

class CExample
{
private:
    int a;
public:
    //构造函数
    CExample(int b)
    {
     a=b;
        printf("constructor is called\n");
    }
    //拷贝构造函数
    CExample(const CExample & c)
    {
     a=c.a;
        printf("copy constructor is called\n");
    }
    //析构函数
    ~CExample()
    {
     cout<<"destructor is called\n";
    }
    void Show()
    {
     cout<<a<<endl;
    }
};


CExample g_fun()
{
    CExample temp(0);
    return temp;
}


int main()
{
    
    g_fun();
    return 0;
}
```

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/3.jpg)

当g_Fun()函数执行到return时，会产生以下几个重要步骤：

(1). 先会产生一个临时变量，就叫XXXX吧。

(2). 然后调用拷贝构造函数把temp的值给XXXX。整个这两个步骤有点像：CExample XXXX(temp);

(3). 在函数执行到最后先析构temp局部变量。

(4). 等g_fun()执行完后再析构掉XXXX对象。

## 浅拷贝与深拷贝

深拷贝和浅拷贝可以简单理解为：**如果一个类拥有资源，当这个类的对象发生复制过程的时候，资源重新分配，这个过程就是深拷贝，反之，没有重新分配资源，就是浅拷贝**。

深拷贝与浅拷贝的区别就在于深拷贝会在堆内存中另外申请空间来储存数据，从而也就解决了指针悬挂的问题。**简而言之，当数据成员中有指针时，必须要用深拷贝**。

### 默认拷贝构造函数

很多时候在我们都不知道拷贝构造函数的情况下，传递对象给函数参数或者函数返回对象都能很好的进行，这是因为编译器会给我们自动产生一个拷贝构造函数，这就是“默认拷贝构造函数”，这个构造函数很简单，仅仅使用“老对象”的数据成员的值对“新对象”的数据成员一一进行赋值，它一般具有以下形式：

```c++
Rect::Rect(const Rect& r)
{
    width=r.width;
    height=r.height;
}
```

### 浅拷贝

所谓浅拷贝，指的是在对象复制时，只对对象中的数据成员进行简单的赋值，默认拷贝构造函数执行的也是浅拷贝。大多情况下“浅拷贝”已经能很好地工作了，但是一旦对象存在了动态成员（例如，在堆中分配内存），那么浅拷贝就会出问题了，让我们考虑如下一段代码：

```c++
#include<iostream>
#include<assert.h>
using namespace std;

class Rect
{
public:
    Rect()
    {
     p=new int(100);
    }
   
    ~Rect()
    {
     assert(p!=NULL);
        delete p;
    }

private:
    int width;
    int height;
    int *p;
};


int main()
{
    Rect rect1;
    Rect rect2(rect1);
    return 0;
}
```

在这段代码运行结束之前，会出现一个运行错误。原因就在于在进行对象复制时，对于动态分配的内容没有进行正确的操作。我们来分析一下：

在运行定义rect1对象后，由于在构造函数中有一个动态分配的语句，因此执行后的内存情况大致如下：

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/4.jpg)

在使用rect1复制rect2时，由于执行的是浅拷贝，只是将成员的值进行赋值，这时 rect1.p = rect2.p，也即这两个指针指向了堆里的同一个空间，如下图所示：

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/5.jpg)

当然，这不是我们所期望的结果，在销毁对象时，两个对象的析构函数将对同一个内存空间释放两次，这就是错误出现的原因。我们需要的不是两个p有相同的值，而是两个p指向的空间有相同的值，解决办法就是使用“深拷贝”。

### 深拷贝

在“深拷贝”的情况下，对于对象中动态成员，就不能仅仅简单地赋值了，而**应该重新动态分配空间**，如上面的例子就应该按照如下的方式进行处理：

```c++
#include<iostream>
#include<assert.h>
using namespace std;

class Rect
{
public:
    Rect()
    {
     p=new int(100);
    }
    
    Rect(const Rect& r)
    {
     width=r.width;
        height=r.height;
     p=new int(100);
        *p=*(r.p);
    }
     
    ~Rect()
    {
     assert(p!=NULL);
        delete p;
    }

private:
    int width;
    int height;
    int *p;
};


int main()
{
    Rect rect1;
    Rect rect2(rect1);
    return 0;
}
```

此时，在完成对象的复制后，内存的一个大致情况如下：

![](http://oklbfi1yj.bkt.clouddn.com/c++%E6%8B%B7%E8%B4%9D%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/6.jpg)

此时rect1的p和rect2的p各自指向一段内存空间，但它们指向的空间具有相同的内容，这就是所谓的“深拷贝”。

### 防止默认拷贝发生

#### 声明一个私有拷贝构造函数

可以不必去定义这个拷贝构造函数，这样因为拷贝构造函数是私有的，如果用户试图按值传递或函数返回该类对象，将得到一个编译错误，从而可以避免按值传递或返回对象。

```c++
//防止按值传递
class CExample 
{ 
private: 
    int a; 
  
public: 
    //构造函数
    CExample(int b) 
    { 
        a = b; 
        cout<<"creat: "<<a<<endl; 
    } 
  
private: 
    //拷贝构造函数，只是声明
    CExample(const CExample& C); 
  
public: 
    ~CExample() 
    { 
        cout<< "delete: "<<a<<endl; 
    } 
  
    void Show () 
    { 
        cout<<a<<endl; 
    } 
}; 
  
//???? 
void g_Fun(CExample C) 
{ 
    cout<<"test"<<endl; 
} 
  
int main() 
{ 
    CExample test(1); 
    //g_Fun(test);   //按值传递将出错
      
    return 0; 
}
```

## 拷贝构造函数的几个细节

1.为什么拷贝构造函数必须是引用传递，不能是值传递？

简单的回答是**为了防止无限的递归调用**。
具体一些可以这么讲：
当 一个对象需要以值方式传递时，编译器会生成代码调用它的拷贝构造函数以生成一个复本。如果类A的拷贝构造函数是以值方式传递一个类A对象作为参数的话，当需要调用类A的拷贝构造函数时，需要以值方式传进一个A的对象作为实参； 而**以值方式传递需要调用类A的拷贝构造函数；结果就是调用类A的拷贝构造函数导致又一次调用类A的拷贝构造函数**，这就是一个无限递归。













