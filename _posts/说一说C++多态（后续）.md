---
title: 说一说C++多态（后续）
date: 2017-11-19 10:10:06
tags:
- C++
---

好久没写博客了，因为在阅读一些书籍，例如《Linux内核设计与实现》、《unix 内核源码剖析》、《剑指offer》（终于A完了《剑指offer》，很happy啊）、《深度探索C++对象模型》，给自己充充电，同时回味一些知识。不能保证都可以吸收，但是挺有收获的，以后肯定还要阅读的。感觉书中有一些东西有些老了，和自己原来所知道的有些不同，但是，主要是学习思想。

OK，开始主题咯。

在上一篇文章《说一说C++多态》中快结尾的部分，我们看到了有这样的两段汇编代码：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/29.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81/26.PNG)

它们分别对应着这两句C++代码：

```c++
pBase->sleep();
pRe.sleep();
```

在那篇文章中，我说就是因为这两段汇编代码实现了多态的效果。有小伙伴对我说，还是有一些懵逼。好的，现在我来多说几句，这两段汇编代码为什么可以实现多态。

首先，我说一说这两段汇编代码真正对应着什么C++代码。也就是说编译器会把：

```c++
pBase->sleep();
pRe.sleep();
```

转化为什么代码。实际上，这两段C++代码都会被转化为一种形式，即：

```
(*ptr->vptr[n])(ptr);
```

这句话中的这些东西代表什么呢？

其中，ptr表示实际指向的类对象的指针（也就是Animal类对象的指针）。

vptr表示由编译器产生的指针（存放在每一个具有虚函数的对象中），指向虚函数表。

n表示整个虚函数在虚函数表中的索引值（这里，n应该为1）。

OK，我补充一些东西帮助大家理解。假如，有一个类是这样定义的：

```c++
#include <iostream>
#include <string>
using namespace std;

class Fruit{
public:
    virtual void Color() {
        cout << "Fruit::color()" << endl;
    }
    virtual void Weight() {
        cout << "Fruit::weight()" << endl;
    }
    virtual void Water() {
        cout << "Fruit::water()" << endl;
    }

private:
	string color;
	double weight;
	double water;
};

int main(int argc, char const *argv[])
{
	Fruit* ptr = new Fruit();

	return 0;
}
```

那么，我可以得到如下的图片：

![](http://oklbfi1yj.bkt.clouddn.com/%E8%AF%B4%E4%B8%80%E8%AF%B4C++%E5%A4%9A%E6%80%81%EF%BC%88%E5%90%8E%E7%BB%AD%EF%BC%89/.1.PNG)

好的，我们再来看看这个：

```c++
(*ptr->vptr[n])(ptr);
```

其中，`ptr`对应着

```c++
Fruit* ptr = new Fruit();
```

中的`Fruit* ptr  `，vptr对应着图中的`_vptr_Fruit`，n对应着`#0`、`#1`......

到这里就快要结束这篇文章了，在这里，有些小伙伴估计发现了一个细节，就是我们在实现多态的时候，使用了指针或者引用：

```c++
Base* pBase = &dargon;
Base& pRe = dog;
```

那么，我们可不可以直接把一个子类对象赋值给父类对象实现多态呢？答案是不能的，为什么？因为这会发生“切割”，因为，一个子类对象所占的内存是大于父类对象所占的内存的，所以把一个子类的对象直接赋值给父类，实际上得到的还是一个父类的对象。所以，实现不了多态。就好比我把一个超出了int型范围的doble类型变量赋值给一个int类型的变量，不能够存下来的，多出来的部分是放不下的。

而指针为什么可以呢？因为指针类型会告诉编译器如何解释某个特定地址中的内容及其大小，然后再配合指向虚函数表的那个`vptr`指针，从而实现多态。

我有一个梦想，happy ending......







