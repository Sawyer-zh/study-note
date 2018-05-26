# 在foreach中利用引用来实现数组的迭代

```php
<?php
$arr = array(1, 2, 3, 4);
foreach ($arr as $key => &$value) {
	# code...
	$value *= 2;
}
echo "now, the array is:\n";
var_dump($arr);
```

此时数组的值为：`array(2, 4, 6, 8)`，**此时$value是数组中最后一个元素的引用**(记住，注意理解！！！)

做个小实验：

```php
<?php
$arr = array(1, 2, 3, 4);
foreach ($arr as $key => &$value) {
	# code...
	$value *= 2;
}
echo "now, the array is:\n";
var_dump($arr);
var_dump($value);
$value = 9;//因为此时的$value是数组的最后一个元素的引用，所以，执行这条语句会把数组的8改成9！
var_dump($arr);
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9C%A8foreach%E4%B8%AD%E5%88%A9%E7%94%A8%E5%BC%95%E7%94%A8%E6%9D%A5%E5%AE%9E%E7%8E%B0%E6%95%B0%E7%BB%84%E7%9A%84%E8%BF%AD%E4%BB%A3/1.PNG)

避免这种错误最好的办法就是在循环后立即用unset函数销毁变量

```php
unset($value);
```



而如果$value不用引用的话，数组的值仍然为：`array(1, 2, 3, 4)`

并且，无论`$value`是引用的方式还是非引用的方式，使用foreach遍历之后，`$value`的值都是`$value`最后的那个值，也就是8