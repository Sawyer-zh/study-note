---
title: 智能指针之-shared_ptr
date: 2017-10-16 18:51:00
tags:
- C++
- Boost
---

![](http://oklbfi1yj.bkt.clouddn.com/%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88%E4%B9%8B-shared_ptr/100.jpg)

### 产生的原因

由于 C++ 语言没有自动内存回收机制，作为程序员的我们每次 new 出来的内存都要手动 delete。而使用智能指针便可以有效缓解这类问题。正所谓多一事不如少一事。

#### 举个例子

```c++
ClassA* temp_ptr = new ClassA();
temp_ptr->foo();
delete temp_ptr;
```

如果我们忘记在调用完temp_ptr之后删除temp_ptr，那么会造成一个悬挂指针，非常容易造成内存泄漏。

除了我们会忘记释放的问题，还有一些问题，比如说我们在前面已经释放过了一次指针。但是，我们写着写着代码，就忘记了之前已经释放过了，然后又释放了一次，造成**二次释放**的问题。为了解决二次释放问题，我们可以这么做：

```C++
ClassA* temp_ptr = new ClassA();
temp_ptr->foo();
delete temp_ptr;
temp_ptr = NULL;
```

上面的问题细心点或许还可以发现。可是事实上，不止是“忘记”，在上述的这一段程序中，**如果foo()在运行时抛出异常，那么temp_ptr所指向的对象仍然不会被安全删除**。

<!-- more -->

### 先来大概认识一下

对于编译器来说，智能指针实际上**是一个栈对象，**智能指针是一个模板，实际上是对普通指针加了一层封装机制，**并非指针类型**，在栈对象生命期即将结束时，**智能指针通过析构函数释放有它管理的堆内存**。这样的一层封装机制的目的是为了使得智能指针可以方便的管理一个对象的生命期。

### 分类

实际上，智能指针有许多种。例如，`auto_ptr`、`shared_ptr`。

在这里，我们重点介绍一下`shared_ptr`。

#### shared_ptr

`shared_ptr`智能指针是采用引用计数的机制实现的。意思是，所有的栈上的内存，在还没有被开辟的时候，该块内存的引用计数为0。如果我有一个裸指针p1，它指向内存m1，当我使用`shared_ptr`智能指针第一次对p1这个裸指针封装了之后，引用计数是1。此时，如果有第二个`shared_ptr`智能指针也想要使用内存m1，则可以把第一个`shared_ptr`智能指针拷贝给第二个`shared_ptr`智能指针，此时，引用计数为2。如果有某个智能指针用完了内存m1，则引用计数减1。显然，当引用计数为0的时候，回到了初始状态。`shared_ptr`智能指针在内部就把裸指针所指的那片内存释放掉。在整个释放过程，我们并没有直接去手动delete，所以说很方便。

#### 实现一个简单的shared_ptr

```c++
template <typename T>
class shared_ptr {
public:
    shared_ptr(T* p) : count(new int(1)), _ptr(p) {}
    shared_ptr(shared_ptr<T>& other) : count(&(++*other.count)), _ptr(other._ptr) {}
    T* operator->() { return _ptr; }
    T& operator*() { return *_ptr; }

    shared_ptr<T>& operator=(shared_ptr<T>& other) {
        ++*other.count;
        
        if (this->_ptr && 0 == --*this->count) {
            delete count;
            delete _ptr;
        }

        this->_ptr = other._ptr;
        this->count = other.count;

        return *this;
    }

    ~shared_ptr() {
        if (--*count == 0) {
            delete count;
            delete _ptr;
        }
    }

    int getRef() { return *count; }

private:
    int* count; // 引用计数
    T* _ptr; // 裸指针，也就是去掉封装之后的那个指针
};
```

（未完，喔去跳舞了:-O）