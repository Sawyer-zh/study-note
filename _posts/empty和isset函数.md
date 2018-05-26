---
title: empty和isset函数
date: 2017-07-04 14:07:35
tags:
- php
---

自己对这两个函数一直是一知半解的，这里小小的总结一下

# empty()函数

对于empty()函数，php手册给出的官方解答是：

### ***判断一个变量是否被认为是空的。当一个变量并不存在，或者它的值等同于FALSE，那么它会被认为不存在。如果变量不存在的话，empty()并不会产生警告***

也就是说**我们可以通过判断与它等价的bool值来判断empty之后的值是false还是true**

我们做一个小实验：

```php
<?php
var_dump(empty(0));
var_dump(!!0);
var_dump(empty(null));
var_dump(!!null);
var_dump(empty(array()));
var_dump(!!array());
var_dump(empty('0'));
var_dump(!!'0');
var_dump(empty(1));
var_dump(!!1);
```

下图是打印出的结果：

![](http://oklbfi1yj.bkt.clouddn.com/hexo/6.PNG)

也就是说，empty()函数里面的参数的值的bool等价值为false的话，那么empty函数返回true。否则返回false

# isset()函数

对于isset()函数，php手册给出的官方解答是：

### ***检测变量是否设置，并且不是 NULL***

变量是否设置比较好理解，也就是给它值就行了(不能是NULL)，那么什么时候一个变量会是null呢？

手册里面给出的答案是：

- ### ***被赋值为 NULL***

- ### ***尚未被赋值***

- ### ***被 unset()***

做个小实验：

```php
<?php
$var1 = '';
var_dump(isset($var1));
$var2 = 0;
var_dump(isset($var2));
$var3 = '0';
var_dump(isset($var3));
$var4 = array();
var_dump(isset($var4));
$var5 = new stdClass();
var_dump(isset($var5));
```

![](http://oklbfi1yj.bkt.clouddn.com/hexo/7.PNG)



