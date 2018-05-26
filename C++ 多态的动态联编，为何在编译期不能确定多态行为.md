# C++ 多态的动态联编，为何在编译期不能确定多态行为

```c++
#include "iostream";
using namespace std;
class Base
{
public:
    int a;
    virtual void print() {
        cout << "我是父类1111111111111" << endl;
    }
    void print2() {
        cout << "我是父类4444444444444444" << endl;
    }
};
class Child :public Base
{
public:
    int b;
    virtual void print() {
        cout << "我是孩子222222222" << endl;
    }
    void print2() {
        cout << "我是孩子33333333333" << endl;
    }
};
void setData(Base *p)
{
    p->print();//0x84000
    //假如调用Base的print（假如print函数的地址为0x84000，也就是在此处调用这个地址的函数）
    //那么好，因为setData已经编译完了（把编译好的方法存到方法区），在24行就是调用0x84000，
    //所以无论实参是什么每次执行到24行，都会调用0x84000的函数
    //所以就无法实现多态，为了实现多态，只能在运行到24行的时候去动态判断，而不是提前编译好调用的函数
    //因为一个函数只能编译成一种形式放到方法区
    p->print2();//0x85000
    //如果不是多态，那么执行到34行，永远都是调用0x85000的函数，所以在编译期就可以定好；
}
void main()
{
    Base b;
    Child c;
    setData(&b);
    setData(&c);
}  
```

一个函数只能编译成一种形式，并存放到方法区了，并不是根据不同的实参，编译成不同版本放到方法区。

- setData（&b）因为实参是基类的对象地址，在编译期，如果把setData（Base *p）函数编译成调用base的print（）函数【**在编译阶段，一个函数只能编译成一种形式，并存放到方法区了，并不是根据不同的实参，编译成不同版本放到方法区**】，那么接下来，调用setData（&c）因为函数已经编译好就是调用base类的print方法，所以就会出错，不会调用子类child的print方法，**所以只能在程序运行时，根据实参来判断调用具体的函数**；
- **因为在编译期无法确定到底调用谁的函数，所以就动态绑定**（通过虚函数实现）。