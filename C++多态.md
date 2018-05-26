# C++多态

多态一般的用法，就是拿一个父类的指针去调用子类中被重写的方法。但我搞不懂为什么要那么做，我们直接在子类中写一个同名的成员函数，从而隐藏父类的函数不就行了么？

一个比较好的回答如下：

将父类比喻为电脑的外设接口，子类比喻为外设，现在我有移动硬盘、U盘以及MP3，它们3个都是可以作为存储但是也各不相同。如果我在写驱动的时候，我用个父类表示外设接口，然后在子类中重写父类那个读取设备的虚函数，那这样电脑的外设接口只需要一个。但如果我不是这样做，而是用每个子类表示一个外设接口，那么我的电脑就必须有3个接口分别来读取移动硬盘、U盘以及MP3。若以后我还有SD卡读卡器，那我岂不是要将电脑拆了，焊个SD卡读卡器的接口上去？

所以，用父类的指针指向子类，是为了面向接口编程。大家都遵循这个接口，弄成一样的，到哪里都可以用，准确说就是“一个接口，多种实现“。

## 对对象使用sizeof()

### 示例代码

```c++
#include <iostream>

using namespace std;

class A {
public:
	void g() {
		cout << "I am A::g()" << endl;
	}
};

int main(int argc, char const *argv[]) {
    A a;

    cout << sizeof(a) <<endl;

	return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/1.png)

这个很好理解，但当我们将函数g()加上virtual之后：

```c++
#include <iostream>

using namespace std;

class A {
public:
	virtual void g() {
		cout << "I am A::g()" << endl;
	}
};

int main(int argc, char const *argv[]) {
    A a;

    cout << sizeof(a) <<endl;

	return 0;
}
```

再看结果会看到：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/2.png)

变成了8。这是因为在后者中变成了虚函数了。

virtual是让子类与父类之间的同名函数有联系，这就是多态性，实现动态绑定。

**任何类若是有虚函数就会比正常的类大一点，所有有virtual的类的对象里面最头上会自动加上一个隐藏的、不让我知道的指针，它指向一张表，这张表叫做vtable，vtable里是所有virtual函数的地址**。

下边来看这样两段代码：

```c++
class Shape {
public:
     Shape();
     virtual  ~Shape();
     virtual void render();
     void move(const pos&);
     virtual void resize();
protected:
     pos center;
};
```

这个类的内存分布是这样的：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/3.png)

就是在成员变量（在这里是center）前有一个**vtable**的指针，它会指向一个table,这个table叫做虚函数表。

```c++
class Shape {
public:
     Shape();
     virtual  ~Shape();
     virtual void render();
     void move(const pos&);
     virtual void resize();
protected:
     pos center;
};

class Ellipse : public Shape{
public:
    Ellipse (float majr, float minr);
    virtual void render();

protected:
    float major_axis;
    float minor_axis;
};
```

Ellipse继承Shape，看一下它的内存分布：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/4.PNG)

这里的vtable不是对象的，而是属于类的，这就是多态的实现机制。

**这样由上面的解释我们来详细讲解一下多态的概念和实现：**

多态（Polymorphism）按字面的意思就是“多种状态”。在面向对象语言中，接口的多种不同的实现方式即为多态。引用Charlie Calverts对多态的描述——多态性是允许你将父对象设置成为和一个或更多的他的子对象相等的技术，赋值之后，**父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作**。简单的说，就是一句话：允许将子类类型的指针赋值给父类类型的指针。其实我看到过一句话：调用同名函数却会因上下文的不同而有不同的实现。我觉得这样更加贴切，还加入了多态三要素：（1）相同函数名  （2）依据上下文  （3）实现却不同

再来看这个例子：

```c++
#include <iostream>

using namespace std;

class A {
public:
	A() : i(10) {}
	virtual void f() { cout << "A::f()" << i << endl; }

	int i;
};

class B : public A {
public:
	B() : j(20) {}
	virtual void f() { cout << "B::f() " << j << endl; }

	int j;
};

int main(int argc, char const *argv[])
{
	A a;
    B b;

    A *p = &b;
    p->f();

	return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/5.PNG)

这时我们执行这个程序，b的f()函数会执行。这里就是多态中的**动态绑定**，本来是基类型的指针却赋予了子类型的对象的地址，这样当运行时才能知道执行哪个f()函数。

修改main函数：

```c++
int main(int argc, char const *argv[])
{
	A a;
    B b;

    A *p = &b;
    p->f();

    a = b;
    a.f();

	return 0;
}
```

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/6.PNG)

可见a.f()的结果输出是不同的。有很多理由说明这个，其一就是通过指针或引用才是动态绑定，通过点运算是不可以的。

多态特性的工作依赖虚函数的定义，在需要解决多态问题的重载成员函数前，加上virtual关键字，那么该成员函数就变成了虚函数，从上例代码运行的结果看，系统成功的分辨出了对象的真实类型，成功的调用了各自的重载成员函数。

## 虚函数的定义要遵循以下重要规则

1、静态成员函数不能是虚函数,因为静态成员函数的特点是不受限制于某个对象。

2、只有类的成员函数才能说明为虚函数，因为虚函数仅适合用与有继承关系的类对象，所以普通函数不能说明为虚函数。

3、内联(inline)函数不能是虚函数，因为内联函数不能在运行中动态确定位置。即使虚函数在类的内部定义，但是在编译的时候系统仍然将它看做是非内联的。

4、**构造函数不能是虚函数**，因为构造的时候，对象还是一片未定型的空间，只有构造完成后，对象才是具体类的实例。 

5、析构函数可以是虚函数,而且通常声名为虚函数。

## c++静态多态（编译时候确定）

### 实际编码操作

```c++
#include <iostream>
using namespace std;

void fun1()
{
    cout << "fun1 call" << endl;
}

void fun1(int a)
{
    cout << "fun1 a call" << endl;
}

int main()
{
    fun1();
    fun1(10);
    return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/7.png)

### 实现的原理

- 通过重载（overload）的特性来实现，**在编译阶段就决定要调用那个函数，故称为静态多态**。
- c++编译器在编译代码时，会对函数符号重签名（c编译器不会），当c++编译器遇到重载调用时则直接调用重签名后的函数，**使用nm命令查看可执行文件的符号我们看到两个被重签名的符号**

### 以c的视角理解

```c
#include <stdio.h>

void _Z4fun1v()
{
    printf("fun1 call\n");
}

void _Z4fun1i(int a)
{
    printf("fun1 a call\n");
}

int main()
{
    _Z4fun1v(); //对应之前的void fun1();
    _Z4fun1i(10); //对应之前的void fun1(int a);
    return 0;
}
```

## c++动态多态（运行时候确定）

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void sleep()
    {
        cout << "Base sleep" << endl;
    }
    virtual void eat()
    {
        cout << "Base eat" << endl;
    }
    virtual void run()
    {
        cout << "Base run" << endl;
    }
};

class Animal : public Base
{
public:
    size_t age;
    
    void sleep()
    {
        cout << "Animal sleep" << endl;
    }
    void eat()
    {
        cout << "Animal eat" << endl;
    }
    void run()
    {
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

int main()
{
    Animal dargon;
    Animal dog;
    
    Base * pBase = &dargon;
    Base & pRe = dog;
    
    //通过基类的指针指向派生类对象来实现动态多态
    pBase->sleep();
    //通过基类的引用指向派生类对象来实现动态多态
    pRe.sleep();
    
    /*
        取出Animal的虚表指针.
        (size_t *)&dargon => dargon起始地址转换为size_t *
        *(size_t *)&dargon => dargon起始地址开始取sizeof(size_t)个字节解析成size_t(虚表指针的值)
        (size_t *)*(size_t *)&dargon => 把这个值转换成size_t *类型
    
        ps: size_t在32位机是4个字节，在64位机是8个字节，指针变量的大小和size_t的大小是一致的。
     */
    size_t * vptable_dargon = (size_t *)*(size_t *)&dargon;
    size_t * vptable_dog = (size_t *)*(size_t *)&dog;
    
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

结果：

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/8.png)

### 实现原理

- **通过c++的重写（override）的特性来实现**，**只有在运行时才知道真正调用的是哪个函数**，**故称为动态（指的是运行时候确定）多态**。
- c++为有虚函数的每个类添加了一个虚函数表（类的静态变量），并在**每个类对象的起始地址处嵌入一个虚表指针**指向虚函数表，再通过这个虚表指针来实现运行时的多态。

![](http://oklbfi1yj.bkt.clouddn.com/C++%E5%A4%9A%E6%80%81/9.jpg)



虚函数表是类所拥有的，程序运行过程中不能够修改，**它存放在常量区**。

**一个类若继承了多个含有虚函数的基类，那么该类就有对应数量的虚函数表**。



