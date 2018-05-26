---
title: 一道java期中考试题目引发的思考
date: 2017-11-27 21:51:25
tags:
- C++
---

今天进行了java的期中考试，发现不翻书真的记不住啊......java和C++的差别还是挺多的（全程带着C++的思维去做的）......

好了，进入正题......

在这次的试题中，大概有这么一段java代码：

```java
class A {
    String s = "class A";
    // other code ......
}

class B extends A {
    String s = "class B";
    // other code ......
}

public class TypeConvert {
    public static void main (String[] args) {
        B b1;
        B b2 = new B();

        A a1, a2;

        a1 = (A)b2;
        b1 = (B)a1;
    }
}
```

在这里，我断章取义了一下，直接看了最后一句：

```java
b1 = (B)a1;
```

我就感觉这是一条把父类对象赋值给子类对象的语句（实际上在java里面这是引用，我以为这是C++中的以静态方式创建的对象）。

然后我就想：在C++中能否也以这样的方式赋值？

于是我写下这样的代码：

```c++
#include <iostream>

using namespace std;

class A {
public:
	string s;
	A() {
		s = "class A";
	}
	virtual void show() {
		cout << s << endl;
	}
};

class B: public A {
public:
	string s;
	B() {
	    s = "class B";
	}

	virtual void show () {
		cout << s << endl;
	}
};

int main(int argc, char const *argv[])
{
	A a1;
	B b1;

	b1 = a1;

	cout << b1.s << endl;
	cout << a1.s << endl;

	
	return 0;
}
```

编译结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%B8%80%E9%81%93java%E6%9C%9F%E4%B8%AD%E8%80%83%E8%AF%95%E9%A2%98%E7%9B%AE%E5%BC%95%E5%8F%91%E7%9A%84%E6%80%9D%E8%80%83/1.PNG)

我们会发现，这里有一句：

```shell
no match for 'operator=' (operand types are 'B' and 'A')
  b1 = a1;
```

的报错。

意思是没有匹配到对应的重载的`=`运算符。

我先说一下我开始的时候是怎么分析的：首先，我们知道父类对象a1所占的内存要比子类对象b1所占的内存小，所以，把一个占用内存更小的父类对象a1拷贝到占用内存更大的子类对象b1应该是可以做到的。也就是说，我是从内存大小的角度去思考这个问题的。然而，在这里我忘了一个致命性的问题：**用一个对象给另一个对象赋值是不行的**。那为什么有时候又可以使用`=`把一个对象给另一个对象赋值呢？因为，编译器帮我们做了一件事情：合成了一个默认的**拷贝构造函数**。

我是看到这个提醒才醒悟过来：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%B8%80%E9%81%93java%E6%9C%9F%E4%B8%AD%E8%80%83%E8%AF%95%E9%A2%98%E7%9B%AE%E5%BC%95%E5%8F%91%E7%9A%84%E6%80%9D%E8%80%83/2.PNG)

这个

```c++
B& B::operator=(const B&)
```

就是编译器为我们偷偷做的事情。

我们再来看下一个提醒：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%B8%80%E9%81%93java%E6%9C%9F%E4%B8%AD%E8%80%83%E8%AF%95%E9%A2%98%E7%9B%AE%E5%BC%95%E5%8F%91%E7%9A%84%E6%80%9D%E8%80%83/3.PNG)

说的是类型转换出了问题。

所以我们修改一下代码：

```c++
b1 = (B&)a1;
```

再来看一下结果：
![](http://oklbfi1yj.bkt.clouddn.com/%E4%B8%80%E9%81%93java%E6%9C%9F%E4%B8%AD%E8%80%83%E8%AF%95%E9%A2%98%E7%9B%AE%E5%BC%95%E5%8F%91%E7%9A%84%E6%80%9D%E8%80%83/4.PNG)

OK，编译通过。

其实这里还有一个疑问，先保留，等我在stackoverflow上提问后，得到解答了再来更新，嘿嘿......



