# JSON的基本语法

1、并列的数据之间用逗号（,）分隔

2、映射用冒号（:）表示

3、并列数据的集合（数组）用方括号（[]）表示

4、映射的集合（对象）用大括号（{}）表示

## 优点

1、数据格式比较简单，易于读写，格式都是压缩的，占用带宽小

2、支持多种语言，包括C，C#，Java，JavaScript，Perl，PHP

## 缺点

1、要求字符集必须是Unicode，受约束性强

2、语法语法过于严谨，必须遵循JSON语法四个原则



## 怎么使用JSON

-JSON数据格式和serialize数据格式的异同和使用

-PHP中操作JSON的重要函数

-一维数组到JSON数据格式的转换

-多维数组到JSON数据格式的转换

-对象到JSON数据格式的转换

​	1、对象转换为JSON数据时，只转换公有变量，私有变量不转换

-如何解析一个JSON数据格式

-转换JSON数据格式到对象类型

​	1、$jsonStr = '{"key":"value", "key1":"value1"}';

​	$jsonArray = json_decode($jsonStr);

​	print_r($jsonArray);

-转换JSON数据格式到数组类型

​	1、php中，把一个json字符串转化为一个数组方法：

​	$jsonStr = '{"key":"value", "key1":"value1"}';

​	$jsonArray = json_decode($jsonStr, true);//注意不能漏了后面第二个参数true，否则会转化为std对象

​	print_r($jsonArray);

## JSON数据格式和serialize数据格式的异同和使用

相同点：

-都是把其他数据类型转换成一个可以传输的字符串

-都是结构性数据

不同点：

-serialize序列化后的数据格式保存数据原有类型

-JSON数据格式要更简洁相比serialize序列化之后的数据格式

使用场景：

-JSON适合数据量大，不要求保留原有数据类型的情况下使用

-serialize适合存储带有加密方式的数据串，防止数据被中途截取反序列化破解