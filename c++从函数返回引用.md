# c++从函数返回引用

## 函数返回值和返回引用是不同的

函数返回**值**时会产生一个临时变量作为函数返回值的副本，而返回**引用**时不会产生值的副本，既然是引用，那引用谁呢？这个问题必须清楚，否则将无法理解返回引用到底是个什么概念。以下是几种引用情况：

1、引用函数的参数，当然该参数也是一个引用

```c++
const string &shorterString(const string &s1,const string &s2) {
    return s1.size()<s2.size()?s1:s2;
}
```

以上函数的返回值是引用类型。无论返回s1或是s2,调用函数和返回结果时，都没有复制这些string对象。简单的说，返回的引用是函数的参数s1或s2，同样s1和s2也是引用，而**不是在函数体内产生的**。**函数体内局部对象是不能被引用的，因为函数调用完局部对象会被释放**。

2、千万不要返回局部对象的引用

```c++
const string &mainip(const string &s) {
    string ret=s;
    
    return ret;
}
```

当函数执行完毕，程序将释放分配给局部对象的存储空间。此时，对局部对象的引用就会指向不确定的内存。

3，在类的成员函数中，返回引用的类对象，当然不能是函数内定义的类对象（会释放掉），**一般为this指向的对象**，典型的例子是**string类的赋值函数**

```c++
String& String::operator=(const String &str)  //注意与“+”比较，函数为什么要用引用呢？a=b=c，可以做为左值  
{
    if (this == &str)
    {
        return *this;
    }
    
    delete [] m_string;
  
    int len = strlen(str.m_string);
    m_string = new char[len+1];
    strcpy(m_string, str.m_string);
  
    
    return *this; // 返回本对象，但是，因为返回值的类型是引用，所以返回出去的是本对象的引用而不是副本
} 
```

这与sting类中的“+”运算符重载不一样。“+”运算符的重载不能返回引用，因为它返回的是在函数内定义的类对象，附上代码。

```c++
String String::operator +(const String &str)      
{  
    String newstring;  
    if (!str.m_string)  
    {  
        newstring = *this;  
    }  
    else if (!m_string)  
    {  
        newstring = str;  
    }  
    else  
    {  
        int len = strlen(m_string)+strlen(str.m_string);  
        newstring.m_string = new char[len+1];  
        strcpy(newstring.m_string,m_string);  
        strcat(newstring.m_string,str.m_string);  
    }  
    return newstring;  
} 
```



