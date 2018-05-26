# Google工程师深度讲解Go语言

## 1、课程导读

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/1.png)

## 2、变量定义

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/2.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/3.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/4.png)

:= 只能在函数内使用，在函数外面不能使用。

## 3、变量类型

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/5.png)

### 强制类型转换

Go语言**只有强制类型转换**，没有隐式类型转换。

## 4、常量

Go语言的常量一般不会去大写。

### 使用常量定义枚举类型

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/6.png)

## 5、控制语句

### 条件

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/7.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/8.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/9.png)

### 循环

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/10.png)

Go语言没有while循环。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/11.png)

## 6、函数

这样是不正确的 Go 代码：

```go
func g()
{
}
```

它必须是这样的：

```go
func g() {
}
```

可以返回多个值。但是一般不建议返回多个值。（一般在返回错误信息的时候，才会返回多个值）

**函数可以将其他函数调用作为它的参数，只要这个被调用函数的返回值个数、返回值类型和返回值的顺序与调用函数所需求的实参是一致的**，例如：

假设 f1 需要 3 个参数 `f1(a, b, c int)`，同时 f2 返回 3 个参数 `f2(a, b int) (int, int, int)`，就可以这样调用 f1：`f1(f2(a, b))`。

**在 Go 里面函数重载是不被允许的。**Go 语言不支持这项特性的主要原因是**函数重载需要进行多余的类型匹配影响性能**。

函数不能在其它函数里面声明（不能嵌套），不过我们可以通过使用匿名函数来破除这个限制。

任何一个有返回值（单个或多个）的函数都必须以 `return` 或 `panic`结尾。

在函数调用时，**像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）**。

**如果一个函数需要返回四到五个值，我们可以传递一个切片给函数（如果返回值具有相同类型）或者是传递一个结构体（如果返回值具有不同的类型）**。因为传递一个指针允许直接修改变量的值，消耗也更少。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/12.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/13.png)

### 可变参数

```go
package main

import "fmt"

func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}

func main() {
    sum(1, 2)
    sum(1, 2, 3)

    nums := []int{1, 2, 3, 4}
    sum(nums...) // 因为nums是一个多参数的切片，所以可以直接传一个 nums...
}
```

打印出结果：

```shell
[1 2] 3
[1 2 3] 6
[1 2 3 4] 10
```

### 空白符

空白符用来**匹配一些不需要的值**，然后**丢弃掉**。

### 内置函数

#### new、make

new 和 make 均是**用于分配内存**：new 用于值类型和用户定义的类型，如自定义结构，**make 用于内置引用类型（切片、map 和管道）**。它们的**用法就像是函数，但是将类型作为参数**：new(type)、make(type)。**new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针（详见第 10.1 节）。它也可以被用于基本类型：`v := new(int)`。make(T) 返回类型 T 的初始化之后的值**，因此它比 new 进行更多的工作（详见第 7.2.3/4 节、第 8.1.1 节和第 14.2.1 节）**new() 是一个函数，不要忘记它的括号**

### 将函数作为参数

函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为回调。

### 闭包

当我们不希望给函数起名字的时候，可以使用匿名函数，例如：`func(x, y int) int { return x + y }`。

这样的一个函数不能够独立存在（编译器会返回错误：`non-declaration statement outside function body`），但可以被赋值于某个变量，即保存函数的地址到变量中：`fplus := func(x, y int) int { return x + y }`，然后通过变量名对函数进行调用：`fplus(3,4)`。

也可以直接对匿名函数进行调用：`func(x, y int) int { return x + y } (3, 4)`。

```go
package main

import "fmt"

func main() {
    var f = Adder()
    fmt.Print(f(1), " - ")
    fmt.Print(f(20), " - ")
    fmt.Print(f(300))
}

func Adder() func(int) int {
    var x int
    return func(delta int) int {
        x += delta
        return x
    }
}
```

我们可以看到，在多次调用中，变量 x 的值是被保留的，即 `0 + 1 = 1`，然后 `1 + 20 = 21`，最后 `21 + 300 = 321`：闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。

#### 使用闭包调试

包 `runtime` 中的函数 `Caller()` 提供了相应的信息，因此可以在需要的时候实现一个 `where()` 闭包函数来打印函数执行的位置：

```go
package main

import (
   "runtime"
   "log"
)

func main() {
   where := func() {
      _, file, line, _ := runtime.Caller(1)
      log.Printf("%s:%d", file, line)
   }
   where()
   // some code
   where()
   // some more code
   where()
}
```

## 7、指针

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/14.png)

### 参数传递

在C++中，参数即可以通过值传递（传指针本质上也是传递值）也可以通过引用传递。

但是Go语言只有值传递。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/15.png)

只有值传递并不意味着效率会下降，因为我们传递的值可以是指针。

## 8、数组

如果我们想**让数组元素类型为任意类型的话可以使用空接口作为类型**。当使用值时我们必须先做一个类型判断。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/17.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/16.png)

arr3中的`...`是让编译器自己来计算数组中有几个元素。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/18.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/19.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/20.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/21.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/22.png)

### 多维数组

Go 语言的多维数组是矩形式的（即内部数组<小数组>总是长度相同的，唯一的例外是切片的数组）

### 将数组传递给函数

把一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：

- 传递数组的指针
- 使用数组的切片

## 9、切片（slice）

切片提供了一个相关数组的动态窗口。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/23.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/24.png)

s1：`[2 3 4 5]`

s2：`[5 6]`

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/25.png)

切片也可以是多维的。但是和多维数组不同，大切片的内层小切片的长度可以不相同。

### Slice的实现

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/26.png)

使用取下标运算符[]只能够取得len中的元素。而slice扩展不仅可以取到len中的值，还可以取到cap中的值。（这种思想和STL中的vector类似）

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/27.png)

### Slice的一些操作

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/28.png)

请注意，我们**需要接受来自append的返回值**，因为我们可能会得到一个新的切片值。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/29.png)

## 10、map

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/30.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/31.png)

```go
    m := make(map[string]int)

    m["k1"] = 7
    m["k2"] = 13

    fmt.Println("map:", m) // 打印出 map: map[k1:7 k2:13]

    delete(m, "k2") // 移除k2对应的键值对
    fmt.Println("map:", m) // 打印出 map: map[k1:7]

    _, prs := m["k2"] // 其实m[key]会返回两个值，第二个是一个bool值，判定map中是否有这个键
    fmt.Println("prs:", prs) // 打印出false
```

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/32.png)

## 11、面向对象

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/33.png)

### 结构体的定义

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/34.png)

### 结构体的创建

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/35.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/36.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/37.png)

### 为结构体定义方法

Go支持在结构类型上定义的方法。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/38.png)

这种定义方式是拷贝了一份TreeNode

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/39.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/40.png)

### 封装

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/41.png)

这里的public和private是针对包来说的。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/42.png)

main函数只能在main包里面。

每个目录下都有一个main。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/43.png)

### 扩展已有类型

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/44.png)

## 12、GOPATH

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/45.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/46.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/47.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/48.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/49.png)

## 13、接口

接口是方法签名的集合。

目前为止，我们看到的类型都是具体的类型。一个具体的类型可以准确的描述它所代表的值，并且展示出对类型本身的一些操作方式：就像数字类型的算术操作，切片类型的取下标、添加元素和范围获取操作。具体的类型还可以通过它的内置方法提供额外的行为操作。总的来说，当你拿到一个具体的类型时你就知道它的本身是什么和你可以用它来做什么。

在Go语言中还存在着另外一种类型：接口类型。接口类型是一种抽象的类型。

### 接口的定义

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/55.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/56.png)

 ![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/50.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/51.png)

### 接口变量里面有什么

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/52.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/53.png)

### 查看接口变量

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/54.png)

### 具体例子

```go
package main

import "fmt"
import "math"

// 接口定义
type geometry interface {
    area() float64
    perim() float64
}

type rect struct {
    width, height float64
}
type circle struct {
    radius float64
}

// 在rect结构体和circle结构体上面实现这两个接口
func (r rect) area() float64 {
    return r.width * r.height
}
func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}

func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}
func (c circle) perim() float64 {
    return 2 * math.Pi * c.radius
}

func measure(g geometry) {
    fmt.Println(g)
    fmt.Println(g.area())
    fmt.Println(g.perim())
}

func main() {
    r := rect{width: 3, height: 4}
    c := circle{radius: 5}

    // circle和rect结构体类型都实现了geometry接口，所以我们可以使用这些结构体的实例作为参数进行调用。
    measure(r) // 很像C++的多态，geometry接口类似父类，rect和circle结构体类似子类
    measure(c)
}
```

输出：

```go
{3 4}
12
14
{5}
78.53981633974483
31.41592653589793
```

## 14、函数式编程

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/61.png)

### 函数与闭包

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/57.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/60.png)

可以看到，adder这个函数里面存了一个sum。sum不是函数体里面定义的，它是函数所处的一个环境，sum是外面的，它叫做自由变量。自由变量呢，编译器就会连一根线，连到sum里面去。在这里，我们的sum是一个int型变量，但是自由变量也可以是一个结构，它可以一直连下去，就如上图所示。我们不断的找这种连接关系，最终会把所有我们需要连接的变量连完。所有东西连到以后，我们把这个整体叫做闭包。

return后面的那个 func 可以看作是一个函数体。

函数体中的value是一个局部变量。





![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/58.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/59.png)

## 15、资源管理与出错处理

### defer调用

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/62.png)

### 何时调用defer

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/63.png)

关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 `finally` 语句块，**它一般用于释放某些已分配的资源**。关键字 defer 允许我们进行一些函数执行完成后的收尾工作，例如：

1、关闭文件流

```go
// open a file  
defer file.Close()
```

2、解锁一个加锁的资源

```go
mu.Lock()  
defer mu.Unlock()
```

3、打印最终报告

```go o
printHeader()  
defer printFooter()
```

4、关闭数据库链接

```go
// open a database connection  
defer disconnectFromDB()
```

## 16、错误处理

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/64.png)

## 17、并发编程

“**顺序通信进程**”(communicating sequential processes)或被简称为CSP。

**CSP是一种现代的并发编程模型**，在这种编程模型中值会在不同的运行实例(goroutine)中传递，尽管大多数情况下仍然是被限制在单一实例中。

### goroutine（go语言的协程）

在Go语言中，每一个并发的执行单元叫作一个goroutine。

当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。

新的goroutine会用go语句来创建。在语法上，go语句是一个普通的函数或方法调用前加上关键字go。go语句会使其语句中的函数在一个新创建的goroutine中运行。而go语句本身会迅速地完成。

```go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

**除了从主函数退出或者直接终止程序之外，没有其它的编程方法能够让一个goroutine来打断另一个的执行**，但是之后可以看到**另一种方式来实现这个目的，通过goroutine之间的通信来让一个goroutine请求其它的goroutine，并让被请求的goroutine自行结束执行**。

 `return` 语句也可以用来结束 for 死循环，或者结束一个协程（goroutine）

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/65.png)

Go语言的main程序一旦退出，那么所有的goroutine就会被杀掉。

实际上我们的main函数自己也是一个goroutine。

### 协程 Coroutine

- 轻量级“线程”

作用和线程看起来差不多，初看起来都是去并发的执行一些任务的。但是它是轻量级的，我们可以看到，我们开了1000个协程也是没有问题的。

- 非抢占式多任务处理，由协程主动交出控制权

线程在任何时候都有可能被操作系统切换。哪怕是一条语句执行到一半，也可能会被切换掉。然后操作系统在某个时候又把这个线程切换回来，继续那条语句。而协程不一样，我什么时候想交出对CPU的控制权，什么时候不想，是由协程内部决定的。正是因为这个非抢占式，才能做到轻量级。而抢占式的话，就要处理最坏的情况：我去抢的话，人家的事情正好做到一半，那么要存更多的上下文信息。而非抢占式只需要处理其中切换的几个点就行了。这样对资源的消耗就会小一点。

- 编译器/解释器/虚拟机层面的多任务

它不是操作系统层面的一个多任务。操作系统还是没有协程，只有线程。在Go语言中，可以看成是编译器级别的多任务，编译器会把我们的`go func`来解释为一个协程。具体在执行上，Go语言会有一个调度器来调度我们的协程。我们操作系统本身有一个调度器，Go语言里面有它自己的调度器来调度我们轻量级的线程，即协程。

- 多个协程可能在一个或多个线程上运行

这个是由调度器来决定的。

但是，在上面的程序，我们感觉协程它是被抢占了，因为打印到一半，又被其他协程抢占了。这是因为printf是一个IO的操作，在IO操作的过程中会进行切换，因为IO的操作会有一个等待的过程。

### 协程主动交出控制权的方法

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/66.png)

### Go语言的调度器

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/67.png)

即协程是比子程序更加宽泛的概念。（我们所有的函数调用都可以看作是一个子程序）

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/68.png)

左边这幅图的意思的，在一个线程内，这个线程内有一个main函数，这个main函数去调用doWork函数，只有doWork函数做完了以后，才会把控制权交还给main函数。

而协程不一样，它也是main和doWork，但是main和doWork之间不是一个单向的箭头。中间有一个双向的通道。main和doWork之间的数据可以双向的进行流通。不止是数据，它的控制权也可以双向的流通。就相当于两个并发执行的线程。而main和doWork它们可能运行在一个线程里面，也可能运行在多个线程里面（至于是哪种情况，就不用程序猿管了，由Go语言的调度器决定）。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/69.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/70.png)

### goroutie可能的切换点

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/71.png)

## 18、channel

**默认情况下**发送和接收会**阻塞**，直到发送者和接收者都准备好。这个特点可以实现让我们在程序结束时等待一个消息，而不必使用任何其他同步。

如果说goroutine是Go语言程序的并发体的话，那么channels则是它们之间的通信机制。一个channel是一个通信机制，它可以让一个goroutine通过它给另一个goroutine发送值信息。**每个channel都有一个特殊的类型，也就是channels可发送数据的类型。一个可以发送int类型数据的channel一般写为chan int**。

和map类似，**channel也对应一个make创建的底层数据结构的引用。当我们复制一个channel或用于函数参数传递时，我们只是拷贝了一个channel引用，因此调用者和被调用者将引用同一个channel对象**。和其它的引用类型一样，channel的零值也是nil。

一个channel有发送和接受两个主要操作，都是通信行为。一个发送语句将一个值从一个goroutine通过channel发送到另一个执行接收操作的goroutine。发送和接收两个操作都使用`<-`运算符。在发送语句中，`<-`运算符分割channel和要发送的值。在接收语句中，`<-`运算符写在channel对象之前。**一个不使用接收结果的接收操作也是合法的**。

```go
ch <- x  // a send statement
x = <-ch // a receive expression in an assignment statement
<-ch     // a receive statement; result is discarded
```

**Channel还支持close操作，用于关闭channel，随后对基于该channel的任何发送操作都将导致panic异常。对一个已经被close过的channel进行接收操作依然可以接受到之前已经成功发送的数据**；如果channel中已经没有数据的话将产生一个零值的数据。

使用内置的close函数就可以关闭一个channel：

```go
close(ch)
```

有一个需要注意的地方就是，有`<-ch `一定要有`ch <- x`。但是有`ch <- x`不一定要有`<-ch `。

### 不带缓存的Channels

一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。

基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。

### 带缓存的Channels

```go
package main

import "fmt"

func main() {
    messages := make(chan string, 2)

    messages <- "buffered"
    messages <- "channel"

    fmt.Println(<-messages)
    fmt.Println(<-messages)
}
```

带缓存的Channel可以不需要相同数目的`<-messages`，但是最多不能超过缓存大小的`<-messages`和缓存大小的`messages <-`

### 串联的Channels（Pipeline）

Channels也可以用于将多个goroutine连接在一起，一个Channel的输出作为下一个Channel的输入。这种串联的Channels就是所谓的管道（pipeline）。下面的程序用两个channels将三个goroutine串联起来，如图8.1所示。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/75.png)

**试图重复关闭一个channel将导致panic异常，试图关闭一个nil值的channel也将导致panic异常**。关闭一个channels还会触发一个广播机制。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/72.png)

我们可以开很多的goroutine，goroutine与goroutine之间的双向通道就是channel。

channel和函数一样都是“一等公民”，即可以作为函数的参数，也可以作为函数的返回值。

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/73.png)

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/74.png)

### Channel基本操作语法

```go
c := make(chan bool) //创建一个无缓冲的bool型Channel
c <- x        //向一个Channel发送一个值
<- c          //从一个Channel中接收一个值
x = <- c      //从Channel c接收一个值并将其存储到x中
x, ok = <- c  //从Channel接收一个值，如果channel关闭了或没有数据，那么ok将被置为false
```

**不带缓冲的Channel**兼具通信和同步两种特性，颇受青睐。

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Begin doing something!")
	c := make(chan bool)
	go func() {
		fmt.Println("Doing something…")
		close(c)
	}()
	<-c
	fmt.Println("Done!")
}
```

**这里main goroutine通过"<-c"来等待sub goroutine中的“完成事件”**，sub goroutine（即go func）**通过close channel促发这一事件**。当然**也可以通过向Channel写入一个bool值的方式来作为事件通知**。**main goroutine在channel c上没有任何数据可读的情况下会阻塞等待**。

（即有两种方式来解除 <- c 导致的阻塞）

### 非阻塞的channel

通道上的基本发送和接收处于阻塞状态。但是，我们可以使用带有默认子句的select来实现非阻塞式发送，接收，甚至是非阻塞式多路选择。

```go
package main

import "fmt"

func main() {
	messages := make(chan string)
	signals := make(chan bool)

	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	default:
		fmt.Println("no message received")
	}

	msg := "hi"
	select {
	case messages <- msg:
		fmt.Println("sent message", msg)
	default:
		fmt.Println("no message sent")
	}
  
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	case sig := <-signals:
		fmt.Println("received signal", sig)
	default:
		fmt.Println("no activity")
	}
}
```

### Closing Channel

关闭channel表示不会再发送任何值。**这对于将完成情况传达给频道的接收器很有用**。

### 迭代 Channel

for和range提供了对基本数据结构的迭代。我们也可以使用这种语法来**迭代从通道接收到的值**。

```go
package main

import "fmt"

func main() {
	queue := make(chan string, 2)
	queue <- "one"
	queue <- "two"
	close(queue)

	// 这个 range 迭代每个元素，因为它是从queue中接收。
    // 因为我们关闭了 queue channel，所以迭代后接收2个元素。
	for elem := range queue {
		fmt.Println(elem)
	}
}
```

```shell
one
two
```

## 19、range

range可以迭代各种数据结构中的元素。

## 20、errors

**按照惯例**，错误是最后一个返回值，并且具有类型错误，一个内置的接口。

```go
func f1(arg int) (int, error) {
    if arg == 42 {
        return -1, errors.New("can't work with 42")
    }

    // 当nil是最后一个返回值时候，代表没有错误。
    return arg + 3, nil
}
```

## 21、分布式爬虫

### 爬虫总体算法

![](http://oklbfi1yj.bkt.clouddn.com/Google%E5%B7%A5%E7%A8%8B%E5%B8%88%E6%B7%B1%E5%BA%A6%E8%AE%B2%E8%A7%A3Go%E8%AF%AD%E8%A8%80/76.png)

















































