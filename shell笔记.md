# shell笔记

## 1、#！

`#!`是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell

```Shell
#!/bin/bash
echo "Hello World !"
```

## 2、执行shell

### 方式1：作为可执行程序

注意，一定要写成 `./test.sh`，而不是`test.sh`，运行其它二进制的程序也一样，**直接写 test.sh，linux 系统会去 PATH 里寻找有没有叫 test.sh 的**，而只有 /bin, /sbin, /usr/bin，/usr/sbin 等在 PATH 里，你的当前目录通常不在 PATH 里，所以写成 test.sh 是会找不到命令的，要**用 ./test.sh 告诉系统说，就在当前目录找**。

### 方式2：作为解释器参数

```shell
/bin/php test.php
```

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

## 3、Shell 变量

### 定义变量

定义变量时，变量名不加美元符号（$，PHP语言中变量需要），如：

```shell
your_name="runoob.com"
```

注意，**变量名和等号之间不能有空格**。

不能使用bash里的关键字（**可用help命令查看保留关键字**）。

除了显式地直接赋值，还**可以用语句给变量赋值**，如：

```shell
for file in `ls /etc`
或
for file in $(ls /etc)
```

以上语句将 /etc 下目录的文件名循环出来。

### 使用变量

**使用一个定义过的变量**，只要**在变量名前面加美元符号**即可，如：

```shell
your_name="qinjx"
echo $your_name
echo ${your_name}
```

变量名外面的花括号是可选的，加不加都行，加花括号是为了**帮助解释器识别**变量的边界：

```shell
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done
```

如果不给skill变量加花括号，写成echo "I am good at $skillScript"，解释器就会把$skillScript当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

**推荐给所有变量加上花括号**。

**已定义的变量，可以被重新定义**，如：

```shell
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```

这样写是合法的，但注意，第二次赋值的时候不能写$your_name="alibaba"，使用变量的时候才加美元符`$`。

### Shell 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号。**单双引号的区别跟PHP类似**。

#### 拼接字符串

```shell
your_name="qinjx"
greeting="hello, "$your_name" !"
```

### Shell 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

类似与C语言，数组元素的下标由0开始编号。

#### 定义数组

在Shell中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

```shell
数组名=(值1 值2 ... 值n)
```

例如：

```shell
array_name=(value0 value1 value2 value3)
```

#### 读取数组

读取数组元素值的一般格式是：

```shell
${数组名[下标]}
```

例如：

```shell
valuen=${array_name[n]}
```

使用@符号可以获取数组中的所有元素，例如：

```shell
echo ${array_name[@]}
```

## 4、Shell 注释

以"#"开头的行就是注释，会被解释器忽略。

sh里没有多行注释，只能每一行加一个#号。

如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，怎么办呢？

每一行加个#符号太费力了，可以把这一段要注释的代码用一对花括号括起来，**定义成一个函数**，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。

## 5、Shell 基本运算符

### 算术运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

例如，两个数相加(**注意使用的是反引号 ` 而不是单引号 '**)：

```shell
#!/bin/bash

val=`expr 2 + 2`
echo "两数之和为 : $val"
```

注意：

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
- 完整的表达式要被  包含，注意这个字符不是常用的单引号，在 Esc 键下边。

### 关系运算符

关系运算符**只支持数字**，不支持字符串，**除非字符串的值是数字**。

### 布尔运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

| !    | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| ---- | ---------------------------------- | ------------------------------------- |
| -o   | 或运算，有一个表达式为 true 则返回 true。         | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a   | 与运算，两个表达式都为 true 才返回 true。         | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

### 逻辑运算符

以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:

| 运算符  | 说明      | 举例                                       |
| ---- | ------- | ---------------------------------------- |
| &&   | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\| | 逻辑的 OR  | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

## 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符  | 说明                      | 举例                    |
| ---- | ----------------------- | --------------------- |
| =    | 检测两个字符串是否相等，相等返回 true。  | [ $a = $b ] 返回 false。 |
| !=   | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z   | 检测字符串长度是否为0，为0返回 true。  | [ -z $a ] 返回 false。   |
| -n   | 检测字符串长度是否为0，不为0返回 true。 | [ -n $a ] 返回 true。    |
| str  | 检测字符串是否为空，不为空返回 true。   | [ $a ] 返回 true。       |

## 6、Shell 流程控制

### if else

#### if

if 语句语法格式：

```shell
if condition
#也就是说满足条件的时候，指向后面的N条语句
then
    command1 
    command2
    ...
    commandN 
fi
```

#### if else

if else 语法格式：

```shell
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

### for 循环

与其他编程语言类似，Shell支持for循环。

for循环一般格式为：

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

### 无限循环

无限循环语法格式：

```shell
while :
do
    command
done
```

或者

```shell
while true
do
    command
done
```

```shell
for (( ; ; ))
```

## 7、Shell 函数

linux shell 可以用户定义函数，然后在shell脚本中可以随便调用。

shell中函数的定义格式如下：

```shell
[ function ] funname [()]

{

    action;

    [return int;]

}
```

- 可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。

```shell
demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

带有return语句的函数：

```shell
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

函数返回值在调用该函数后通过 $? 来获得。

注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

### 函数参数

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

```shell
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

```shell
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```

从上面的输出可以看出，`$10` 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用`${n}`来获取参数。



