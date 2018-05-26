# C/C++语言函数

### 1、数据复制

```c
#include <string.h>
void *memcpy(void*dest, const void *src, size_t n);
```

由src指向地址为起始地址的连续n个**字节**的数据复制到以destin指向地址为起始地址的空间内

```c
memcpy(b，a，sizeof (int) *k);
memcpy(b，a，sizeof (double) *k);
```

### 2、把数组元素都初始化同一个值

```c
#include <string.h>
void *memset(void *s, int c, size_t n)
```

将已开辟内存空间 s 的首 n 个字节的值设为值 c

```c
memset (a，0，sizeof (a))
```

### 3、输出到屏幕

printf 输出到屏幕

多数情况下，屏幕总是可以输出的

需要注意的是，`printf `函数在处理变量参数的时候是按照从右至左的次序（因为**C语言的参数是从后往前被堆积在栈中的**），并且会把结果放在栈中。输出的时候，后入栈的结果最先输出来，所以会有一种奇怪的现象：

```c
int count = 0;
printf("%d, %d, %d\n", count++, count++, count++);
```

输出：

```shell
2, 1, 0
```

因为，从最右边开始把结果压栈，所以，是分别把0，1，2压栈。所以出栈的顺序是2，1，0，因此，输出的时候就是2，1，0

为了避免这种讨厌的情况，可以在编译的时候是用 `-Wall` 选项，此时会有警告：

![](http://oklbfi1yj.bkt.clouddn.com/C%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0/1.PNG)

说的是有两个`count`没有定义

### 4、输出到文件

fprintf 输出到文件

文件一般也能写（除非磁盘满或者硬件损坏）

### 5、输出到字符串

```c
int sprintf(char *buffer, const char *format [, argument,...]);
```

例子：

```c
char str[20];
double f=14.309948;
sprintf(str,"%6.2f",f);
```

sprintf 输出到字符串，在我们完成**其他数据类型转换成字符串类型**的操作中应用广泛

字符串就不一定了，应该**保证写入的字符串有足够的空间**，可以容纳输出信息 

### 6、判断是否为英文字母

```c
#include <ctype.h>
int isalpha(int ch);
```

判断字符 ch 是否为英文字母，当ch为英文字母a-z或A-Z时，返回非零值(不一定是1)，否则返回零

### 7、计算三角形的斜边长

```c
#include <math.h>
double hypot(double x, double y); // 其中的 x，y 是两条直角边
```

### 8、排序算法 

```c++
#include<algorithm> 
sort(begin, end);//默认是从小到大排序,begin表示要排序元素的首地址，end表示要排序元素的结束地址。注意，这个函数是左闭右开的[begin, end)，所以，它排序的应该是[begin, end - 1]
```

```
#include<algorithm>
sort(a, a+n, cmp);
```

```c++
int a[n];
sort(a, a+n, cmp);
```

### 9、删除有序数组中重复的元素

删除**有序**数组中的重复元素 

```
unique();
```

### 10、字符串反转

```c++
void reverse(BidirectionalIterator _First, BidirectionalIterator _Last);
```

例如：

```c++
reverse(s1.begin(), s1.end());
```

那么，会对整个`s1`字符串进行反转

### 11、gets()函数接收字符串

```c
char string[15];
gets(string); /*遇到回车认为输入结束*/
```

### 12、计算字符个数

```c++
char a1[100];
int a1_length;
gets(a1); //输入字符串（每输入一个字符，就赋值给一个数组元素）
a1_length = strlen(a1); //计算的字符数组中字符的个数
```

### 13、assert()

如果我要求函数按照我想要的条件执行，可以使用`assert()`这个函数

例如：

```c++
#include <cassert>
#include <ctime>
#include <cstdlib>

// 生成一个随机范围为[rangeL, rangeR]的数
int* generateRandomArray(int n, int rangeL, int rangeR)
{
    assert(rangeL <= rangeR); // 因为我希望rangeL要小于rangeR，所以，执行这一步
  	srand(time(NULL)); // 初始化随机种子
    int num = rand() % (rangeR - rangeL + 1) + rangeL; // 产生随机数
}
```

如果不符合断言中的条件，则程序会终止执行

### 14、交换两个变量的值

```c++
#include <algorithm>
swap(a, b); //交换变量a，b的值
```

### 15、计算程序执行时间

```c++
#include <ctime>

clock_t startTime = clock(); // 返回时钟周期
/*
there are some codes
*/
clock_t endTime = clock(); // 然后用 double(endTime - startTime) / CLOCKS_PRE_SEC 即可得到中间那段代码执行的时间（秒）。其中 CLOCKS_PRE_SEC 表示每秒经过的时钟周期
```

### 16、fprintf

C语言把文件看作一个字符（字节）的序列，即由一个一个字符（字节）的数据顺序组成。根据数据的组织形式，可分为ASCII文件和二进制文件。**ASCII文件又称为文本（text）文件，它的每个字节放一个ASCII代码，代表一个字符。二进制文件是把内存中的数据按其在内存中的存储形式原样输出到磁盘上存放**

#### 用法

```c
/*
fprintf(文件指针,格式字符串,输出表列);
*/
fprintf(fp, "%d", buffer); // 将格式化的数据写入文件
```

例如：

```c
fprintf(stderr, "usage: %s <port>\n", argv[0]);
```

#### 对于stderr解释

默认向屏幕输出

### 17、把数字转换为字符串

#### 方法1

```c++
std::to_string()
```

更多内容见：[C++数值类型与string的相互转换](http://blog.csdn.net/k346k346/article/details/50927002)

#### 方法2

```c++
#include <iostream>

using namespace std;

int main(int argc, char const *argv[])
{
	int i = 1;
	cout<< char(i + '0')<<endl;

	return 0;
}
```

将会输出字符1。

实际上`1 + '0'`会得到整数49。但是使用`char(i + '0')`是可以得到整数代表的ASCII的那个字符

### 18、把字符串转整数

```c++
/*
这是C++ 11的函数，如果不是C++ 11 可以用atoi()
std::stoi()
一般只用第一个参数，后面两个用默认的即可
*/
#include <string>
int stoi (const string&  str, size_t* idx = 0, int base = 10); // 默认是转为10进制
```

解析str将其内容解释为指定基数的整数，它作为int值返回。

### 19、字符串忽略大小写比较

```c
int strcasecmp(const char *s1, const char *s2);
```

### 20、比较字符串前n个字符

```c
/*
返回值：如果str1等于str2，返回值就=0
       如果str1小于str2，返回值就<0
       如果str1大于str2，返回值就>0
*/
int strncmp(char* s1, char* s2, int n);
```

### 21、判断字符是否为字母或数字

```c++
int isalnum (int c);
```

#### 示例代码

```c++
#include <iostream>
#include <ctype.h>

using namespace std;

int main(int argc, char const *argv[])
{
	cout << isalnum('b') << endl;

	cout << isalnum('N') << endl;

	cout << isalnum('9') << endl;

	cout << isalnum(9) << endl;

	cout << isalnum('*') << endl;
	
	return 0;
}
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/C%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0/2.png)

## 22、字符串截取

```c++
/*
参数1：默认的 0 代表从字符串的第一个字符开始截取
参数2：默认的 npos 的值表示直到字符串结尾的所有字符
*/
string substr(size_t pos = 0, size_t len = npos) const;
```



























































































