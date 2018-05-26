# java入门笔记

### 1、编译以及运行java程序

命令行中执行这两步：

```shell
javac HelloWorld.java
#执行完这条命令之后，会出现一个HelloWorld.class文件（这是字节码程序）
#如果注释里面有中文，则执行命令：javac -encoding utf-8 Puppy.java （如果不想每次都敲那么长的命令，可以取一个别名，例如：alias javac='javac -encoding utf-8'）
java HelloWorld
```

### 2、基本语法

#### 大小写敏感

标识符Hello与hello是不同的

#### 类名

类名的首字母应该大写

#### 方法名

以小写字母开头，后面的每个单词首字母大写

#### 源文件名

源文件名必须和类名相同（唯一的public类；切记Java是大小写敏感的），文件名的后缀为.java

#### 主方法入口

所有的Java 程序由**public static void main(String []args)**方法开始执行

### 3、Java标识符

应该以字母（A-Z或者a-z）,美元符（$）、或者下划线（_）作为首字符

首字符之后可以是字母（A-Z或者a-z）,美元符（$）、下划线（_）或数字的任何字符组合

标识符是大小写敏感的

### 4、Java变量

**局部变量**：

在方法、构造方法或者语句块中定义的变量被称为局部变量。变量声明和初始化都是在方法中，**方法结束后，变量就会自动销毁**（局部变量是在栈上分配的

**类变量**（静态变量）：

类变量也声明在类中，方法体之外，但必须声明为static类型

静态变量可以通过：`ClassName.VariableName`的方式访问（但是本类的主函数是可以不需要通过类名来使用静态变量，而其他类想要访问另一个类的静态变量，要使用类名才行）

**成员/实例变量**（非静态变量）：

成员变量是定义在类中，方法体之外的变量。这种变量在创建对象的时候实例化。成员变量可以被类中方法（**主函数不算类方法！！**）、构造方法和特定类的语句块访问

实例变量在对象创建的时候创建，在对象被销毁的时候销毁

### 5、Java数组

数组是储存在堆上的对象，可以保存多个同类型变量

### 6、Java 源程序与编译型运行区别

![](http://oklbfi1yj.bkt.clouddn.com/java%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/1.png)

### 7、Java 对象和类

#### 构造方法

一个类可以有多个构造方法：

```java
public class Puppy{
    public Puppy(){
    }
 
    public Puppy(String name){
        // 这个构造器仅有一个参数：name
    }
}
```

#### 访问实例变量和方法

实例化对象：`ObjectReference = new Constructor();`
访问其中的变量：`ObjectReference.variableName;`
访问类中的方法：`ObjectReference.MethodName();`

#### 源文件声明规则

当在一个源文件中定义多个**外部类**，并且还有import语句和package语句时，要特别注意这些规则：

1、一个源文件中只能有一个public类

2、一个源文件可以有多个非public类

3、源文件的名称应该和public类的类名保持一致

4、如果一个类定义在某个包中，**package语句应该在源文件的首行**

5、如果源文件包含**import语句，应该放在package语句和类定义之间**。如果没有package语句，那么import语句应该在源文件中最前面

6、import语句和package语句对源文件中定义的所有类都有效。在同一源文件中，不能给不同的类不同的包声明

#### Java包（package ）

对类和接口进行分类

package 的作用就是 php 的 namespace 的作用，防止名字相同的类产生冲突

使用例子：

```java
package com.imooc.concurrent;

class Actor extends Thread {
	public void run() {
		System.out.println(getName() + "是一个演员");
		int count = 0;

		System.out.println(getName() + "登台演出" + (++count) + "次");

		System.out.println(getName() + "的演出结束了！");
	}

	public static void main(String[] args) {
		Thread actor = new Actor();
		actor.setName("Mr.Thread");

		actor.start();
	}
}
```

这里，我们第一条语句是`package com.imooc.concurrent;`。也就是说，我这段代码是在包`com.imooc.concurrent;`下面的

然后我们编译：

```shell
javac -d . Actor.java
#其中 -d 后面的 . 代表从当前目录
```

编译后，就会在当前的目录下生成一个目录`com/imooc/concurrent`。在这个目录里面，有一个我们编译好的文件`Actor.class`

执行这个文件：

```shell
java com.imooc.concurrent.Actor
```

**注意**，此时我们所在的目录不是在`com/imooc/concurrent`，而是在包含`com/imooc/concurrent`这个目录的路径下

#### Import语句

package 的作用就是 php 的 use 的作用，防止名字相同的类产生冲突

提供一个合理的路径，使得编译器可以找到某个类

```java
import java.io.*; //命令编译器载入java_installation/java/io路径下（虚拟路径？）的所有类
```

### 8、把类写在多个文件中

java因强制要求类名（唯一的public类）和文件名统一，因此在引用其它类时无需显式声明（也就是不需要类似于php的include）。在编译时，编译器会根据类名去寻找同名文件

### 9、基本数据类型

#### 字符

char类型是一个单一的 16 位 Unicode 字符；

最小值是 \u0000（即为0）；

最大值是 \uffff（即为65,535）；

char 数据类型可以储存任何字符；

例子：char letter = 'A';。

**注意：'-1'是两个字符**

### 10、引用类型

引用类型指向一个对象，指向对象的变量是引用变量。这些变量在声明时被指定为一个特定的类型，比如 Employee、Puppy 等。变量一旦声明后，类型就不能被改变了

对象、数组都是引用数据类型

所有引用类型的默认值都是null

一个引用变量可以用来引用与任何与之兼容的类型

```java
Site site = new Site("Runoob")
```

### 11、Java常量

```java
final double PI = 3.1415927; //使用 final 关键字修饰常量
```

虽然常量名也可以用小写，但为了便于识别，通常使用大写字母表示常量

### 12、自动类型转换

整型、实型（常量）、字符型数据可以混合运算。运算中，不同类型的数据先转化为同一类型，然后进行运算

转换**从低级到高级**（注意是**自动**！！）

> 关于java自动类型的转换有这样一个形象的比喻，当一个小的容器的水换到一个大的容器中毫无问题，但是一个大的容器的水换成小的容器则会装不下，就会溢出

![](http://oklbfi1yj.bkt.clouddn.com/java%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/2.jpg)

如果一个字符自动转换为一个整型，那么整型的值是这个字符的ASCII值

**注意：**

1、不能对boolean类型进行类型转换

2、不能把对象类型转换成不相关类的对象

3、在把容量大的类型转换为容量小的类型时必须使用**强制类型转换**

4、转换过程中可能导致溢出或损失精度，例如：

```java
int i =128;   
byte b = (byte)i; // 此时b是-128
```

5、浮点数到整数的转换是通过舍弃小数得到，而不是四舍五入，例如：

```java
(int)23.7 == 23;		
(int)-45.89f == -45
```

### 13、强制类型转换

```java
(type)value
```

#### 隐含强制类型转换

1、整数的默认类型是 int。

2、浮点型不存在这种情况，因为在定义 float 类型时必须在数字后面跟上 F 或者 f。

### 14、访问控制修饰符

![](http://oklbfi1yj.bkt.clouddn.com/java%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/3.PNG)

#### 访问控制和继承

继承的规则：

1、父类中声明为 public 的方法在子类中也必须为 public。

2、父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public，不能声明为 private。

3、父类中声明为 private 的方法，不能够被继承。

#### static 修饰符

**静态变量**

static 关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝

局部变量不能被声明为 static 变量

**静态方法**

static 关键字用来声明独立于对象的静态方法。静态方法不能使用类的非静态变量

#### final 修饰符

##### final 方法

类中的 final 方法可以被子类继承，但是不能被子类修改

声明 final 方法的主要目的是防止该方法的内容被修改

##### final 类

final 类不能被继承

#### abstract 修饰符

##### 抽象类

**不能用来实例化对象**，声明抽象类的唯一目的是为了将来对该类进行扩充

一个类不能同时被 abstract 和 final 修饰。如果一个类包含抽象方法，那么该类一定要声明为抽象类

抽象类可以包含**抽象方法和非抽象方法**

##### 抽象方法

抽象方法是一种没有任何实现的方法，该方法的的具体实现由子类提供。

抽象方法不能被声明成 final 和 static。

任何继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类。

如果一个类包含若干个抽象方法，那么该类必须声明为抽象类。抽象类可以不包含抽象方法。

抽象方法的声明以分号结尾，例如：**public abstract sample();**

### 15、循环结构

#### 增强型 for 循环

主要用于数组：

```java
int [] numbers = {10, 20, 30, 40, 50};
for(int x : numbers ){
    System.out.print( x );
    System.out.print(",");
}

String [] names ={"James", "Larry", "Tom", "Lacy"};
for( String name : names ) {
    System.out.print( name );
    System.out.print(",");
}
```

### 16、Math类

Math 的方法都被定义为 static 形式，通过 Math 类可以在主函数中直接调用

```java
public class Test {  
    public static void main (String []args)  
    {  
        System.out.println("90 度的正弦值：" + Math.sin(Math.PI/2));  
        System.out.println("0度的余弦值：" + Math.cos(0));  
        System.out.println("60度的正切值：" + Math.tan(Math.PI/3));  
        System.out.println("1的反正切值： " + Math.atan(1));  
        System.out.println("π/2的角度值：" + Math.toDegrees(Math.PI/2));  
        System.out.println(Math.PI);  
    }  
}
```

### 17、Character 类

使用Character的构造方法创建一个Character类对象：

```java
Character ch = new Character('a');
```

### 18、String 类

#### 创建字符串

```java
String greeting = "菜鸟教程";
```

**注意:**String 类是不可改变的，所以你一旦创建了 String 对象，那它的值就无法改变了

#### 字符串长度

```java
String site = "www.runoob.com";
int len = site.length();
```

### 19、StringBuffer 和 StringBuilder 类

当对字符串进行修改的时候，需要使用 StringBuffer 和 StringBuilder 类

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象

它和 StringBuffer 之间的最大不同在于 **StringBuilder 的方法不是线程安全的（不能同步访问）**

由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类

```php
public class Test{
  public static void main(String args[]){
    StringBuffer sBuffer = new StringBuffer("菜鸟教程官网：");
    sBuffer.append("www");
    sBuffer.append(".runoob");
    sBuffer.append(".com");
    System.out.println(sBuffer);  
  }
}

//输出结果：菜鸟教程官网：www.runoob.com
```

### 20、Java 数组

#### 声明数组变量

```java
dataType[] arrayRefVar;   // 首选的方法
```

#### 创建数组

```java
arrayRefVar = new dataType[arraySize];
/*
* 一、使用 dataType[arraySize] 创建了一个数组
* 二、把新创建的数组的引用（地址。地址一般是用16进制表示，而地址的位数取决于cpu是几位的）赋值给变量 arrayRefVar
*/
```

数组变量的声明，和创建数组可以用一条语句完成：

```java
dataType[] arrayRefVar = new dataType[arraySize];
```

还可以使用如下的方式创建数组：

```java
dataType[] arrayRefVar = {value0, value1, ..., valuek};
```

#### 多维数组

```java
String str[][] = new String[3][4];
```

### 21、日期时间

`java.util` 包提供了 Date 类来封装当前的日期和时间

### 22、Java 方法

例如，经常使用到 **System.out.println()**

println() 是一个方法

System 是系统类。

out 是标准输出对象

综合起来就是：调用系统类 System 中的标准输出对象 out 中的方法 println()

#### 方法的命名规则

1、必须以字母、'_'或'＄'开头

2、可以包括数字，但不能以它开头

#### 方法的定义

```java
修饰符 返回值类型 方法名(参数类型 参数名){
    ...
    方法体
    ...
    return 返回值;
}
```

**方法名：**是方法的实际名称。方法名和参数表共同构成方法签名（也就是用来标识/区别一个方法）

#### 方法的重载

如果先定义了一个返回整型最大值的函数

```java
public static int max(int num1, int num2) {
  if (num1 > num2)
    return num1;
  else
    return num2;
}
```

但如果还想得到两个浮点类型数据的最大值，可以使用方法重载：

```java
public static double max(double num1, double num2) {
  if (num1 > num2)
    return num1;
  else
    return num2;
}
```

如果你调用max方法时传递的是int型参数，则 int型参数的max方法就会被调用

如果传递的是double型参数，则double类型的max方法体会被调用，这叫做方法重载

就是说一个类的两个方法拥有相同的名字，但是有不同的参数列表（可以说方法中的参数的类型决定了方法返回值的类型，所以，我们可以不通过方法的返回值类型来标识一个方法）

**Java编译器根据方法签名判断哪个方法应该被调用**

**重载的方法必须拥有不同的参数列表。你不能仅仅依据修饰符或者返回类型的不同来重载方法**

#### 构造方法

Java自动提供了一个默认构造方法，它把所有成员初始化为0

#### 可变参数

一个方法中**只能指定一个可变参数**，它必须是方法的**最后一个参数**。任何普通的参数必须在它之前声明。因为参数个数不定，所以当其后边还有相同类型参数时，java无法区分传入的参数属于前一个可变参数还是后边的参数，所以只能让可变参数位于最后一项

**适用于参数个数不确定，类型确定的情况，java把可变参数当做数组处理**

### 23、变量作用域

局部变量的作用范围从声明开始，直到包含它的块结束

方法的参数范围涵盖整个方法。参数实际上是一个局部变量

for循环的初始化部分声明的变量，其作用范围在整个循环

循环体内声明的变量其适用范围是从它声明到循环体结束

### 24、命令行参数

命令行的参数是传递给main()函数的`args[]`参数的

```java
public class CommandLine {
   public static void main(String args[]){ 
      for(int i=0; i<args.length; i++){
         System.out.println("args[" + i + "]: " +
                                           args[i]);
      }
   }
}
```

结果：

```shell
$ javac CommandLine.java 
$ java CommandLine this is a command line 200 -100
args[0]: this
args[1]: is
args[2]: a
args[3]: command
args[4]: line
args[5]: 200
args[6]: -100
```

### 25、流(Stream)、文件(File)和IO

Java.io 包几乎包含了所有操作输入、输出需要的类

一个流可以理解为一个数据的序列。输入流表示从一个源读取数据，输出流表示向一个目标写数据

#### 读取控制台输入

```java
BufferedReader br = new BufferedReader(new 
                      InputStreamReader(System.in));
```

BufferedReader 对象创建后，我们便可以使用 read() 方法从控制台读取**一个**字符，或者用 readLine() 方法读取一个字符串

#### 从控制台读取多字符输入（即循环输入一个字符）

```java
// 使用 BufferedReader 在控制台读取字符
 
import java.io.*;
 
public class BRRead {
  public static void main(String args[]) throws IOException
  {
    char c;
    // 使用 System.in 创建 BufferedReader 
    BufferedReader br = new BufferedReader(new 
                       InputStreamReader(System.in));
    System.out.println("输入字符, 按下 'q' 键退出。");
    // 读取字符
    do {
       c = (char) br.read();
       System.out.println(c);
    } while(c != 'q');
  }
}
```

#### 从控制台读取字符串

注意：读取多个字符和读取字符串是有区别的

```java
// 使用 BufferedReader 在控制台读取字符
import java.io.*;
public class BRReadLines {
  public static void main(String args[]) throws IOException
  {
    // 使用 System.in 创建 BufferedReader 
    BufferedReader br = new BufferedReader(new
                            InputStreamReader(System.in));
    String str;
    System.out.println("Enter lines of text.");
    System.out.println("Enter 'end' to quit.");
    do {
       str = br.readLine();
       System.out.println(str);
    } while(!str.equals("end"));
  }
}
```

#### 控制台输出

```java
import java.io.*;
 
// 演示 System.out.write().
public class WriteDemo {
   public static void main(String args[]) {
      int b; 
      b = 'A';
      System.out.write(b);
      System.out.write('\n');
   }
}
```

> write() 方法不经常使用，因为 print() 和 println() 方法用起来更为方便

#### 读写文件（创建文件）

##### FileInputStream

该流用于从文件读取数据，它的对象可以用关键字 new 来创建

```java
InputStream f = new FileInputStream("C:/java/hello");
```

也可以使用一个文件对象来创建一个输入流对象来读取文件。我们首先得使用 File() 方法来创建一个文件对象：

```java
File f = new File("C:/java/hello");
InputStream out = new FileInputStream(f);
```

##### FileOutputStream

该类用来创建一个文件并向文件中写数据

如果该流在打开文件进行输出前，目标文件不存在，那么该流会创建该文件

```java
OutputStream f = new FileOutputStream("C:/java/hello")
```

或

```java
File f = new File("C:/java/hello");
OutputStream f = new FileOutputStream(f);
```

### 26、目录

#### 创建目录

**mkdir( )**方法创建一个文件夹，成功则返回true，失败则返回false。失败表明File对象指定的路径已经存在（也就是**目录已经存在**），或者由于整个路径还不存在（也就是**错误的路径**），该文件夹不能被创建

**mkdirs()**方法创建一个文件夹和它的所有父文件夹

```java
import java.io.File;
 
public class CreateDir {
  public static void main(String args[]) {
    String dirname = "codeDir";
    File d = new File(dirname);
    // 现在创建目录
    d.mkdirs();
  }
}
```

执行代码后，会在当前目录创建一个 codeDir 目录

注意：如果`codeDir `目录已经存在了，那么就不会再创建这个目录

#### 读取目录

**一个目录其实就是一个 File 对象**（所以这个 File 对象可以调用一些方法），它包含其他文件和文件夹

#### 删除目录或文件

删除文件可以使用 **java.io.File.delete()** 方法

```java
f.delete();
```

### 27、Scanner 类

Scanner 类（java.util.Scanner）可以用来获取用户的输入

#### 创建 Scanner 对象

```java
Scanner s = new Scanner(System.in);
```

通过 Scanner 类的 **next() 与 nextLine() 方法获取输入的字符串**（程序执行到这里的时候，开始输入和读取字符串了），在读取前我们一般需要 使用 hasNext 与 hasNextLine 判断是否还有输入的数据（程序执行到这里，可以输入字符串，但是不会读取）

##### next 方法

```java
import java.util.Scanner; 
 
public class ScannerDemo {  
    public static void main(String[] args) {  
        Scanner scan = new Scanner(System.in); 
    // 从键盘接收数据  
 
    //nextLine方式接收字符串
        System.out.println("nextLine方式接收：");
        // 判断是否还有输入
        if(scan.hasNextLine()){   
          String str2 = scan.nextLine();
          System.out.println("输入的数据为："+str2);  
        }  
 
    }  
}
```

当我们输入：`runoob com`的时候，会输出`runoob`（会发现com没有被读取到）

##### next() 与 nextLine() 区别

next()：

1、一定要读取到有效字符后才可以结束输入。

2、对输入有效字符之前遇到的空白，next() 方法会自动将其去掉。

3、**只有输入有效字符后才将其后面输入的空白作为分隔符或者结束符**。

4、next() 不能得到带有空格的字符串

nextLine()：

1、**以Enter为结束符,也就是说 nextLine()方法返回的是输入回车之前的所有字符**。

2、可以获得空白。

### 28、继承

```java
class 父类 {
}
 
class 子类 extends 父类 {
}
```

#### 为什么需要继承

如果代码存在重复，导致的后果就是代码量大且臃肿，而且维护性不高(维护性主要是后期需要修改的时候，就需要修改很多的代码，容易出错)，所以要从根本上解决这两段代码的问题，就需要继承，将两段代码中相同的部分提取出来组成 一个父类。子类就不会存在重复的代码，维护性也提高，代码也更加简洁，提高代码的复用性（复用性主要是可以多次使用，不用再多次写同样的代码）

#### 继承的特性

1、子类拥有父类非private的属性，方法

2、子类可以拥有自己的属性和方法，即子类可以对父类进行扩展。

3、子类可以用自己的方式实现父类的方法

4、Java的继承是单继承，但是可以多重继承，单继承就是一个子类只能继承一个父类，多重继承就是，例如A类继承B类，B类继承C类，所以按照关系就是C类是B类的父类，B类是A类的父类

5、提高了类之间的耦合性（继承的缺点，**耦合度高就会造成代码之间的联系密切**）

#### 继承关键字

##### extends关键字

在 Java 中，类的继承是单一继承，也就是说，**一个子类只能拥有一个父类，所以 extends 只能继承一个类**

##### implements关键字

使用范围为类继承接口的情况，可以同时继承多个接口：

```java
public interface A {
    public void eat();
    public void sleep();
}
 
public interface B {
    public void show();
}
 
public class C implements A,B {
}
```

##### super 与 this 关键字

super关键字：引用当前对象的父类。

this关键字：引用自己

##### final关键字

把类定义为不能继承

```java
修饰符(public/private/default/protected) final 返回值类型 方法名(){//方法体}
```

实例变量也可以被定义为 final，被定义为 final 的变量不能被修改。**被声明为 final 类的方法自动地声明为 final，但是实例变量并不是 final**

#### 构造器

子类不能继承父类的构造器（构造方法或者构造函数），但是父类的构造器带有参数的，则必须在子类的构造器中显式地通过super关键字调用父类的构造器并配以适当的参数列表

如果父类有无参构造器，则在子类的构造器中用super调用父类构造器不是必须的，如果没有使用super关键字，系统会自动**调用父类的无参构造器**。所以，无论如何，都会调用父类的构造函数，至于是调用父类的哪一个构造函数，看具体情况

**注意：父类一定要先定义一个无参的构造函数才能去定义一个有参的构造函数。但是，如果不需要实例化父类，那么父类可以没有构造函数**

如果在实例化一个类的时候，没有传递参数过去，则在类中可以不显式定义构造函数

### 29、线程（Thread）

举个例子：

```java
class Actor extends Thread {
	public void run() {
		System.out.println(getName() + "是一个演员");
		int count = 0;

		System.out.println(getName() + "登台演出" + (++count) + "次");

		System.out.println(getName() + "的演出结束了！");
	}

	public static void main(String[] args) {
		Thread actor = new Actor();
		actor.setName("Mr.Thread"); // 设置线程的名字

		actor.start();
	}
}
```

其中：

不论是继承Tread类创建线程还是实现Runnable接口创建线程，**启动线程一般都是调用Thread类的start()方法**，然后由**虚拟机自动调用**Thread类的**run()方法**

`setName`是用来**设置线程的名字**的（setName是Thread里面的方法）

`getName`是用来**获取线程的名字**的（getName是Thread里面的方法）











