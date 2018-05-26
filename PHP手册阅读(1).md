# PHP手册阅读

## 1、类和对象

### 简介

PHP 对待对象的方式与引用和句柄相同，即每个变量都持有对象的引用，而不是整个对象的拷贝。

### 基本概念

#### class

当一个方法在类定义内部被调用时，有一个可用的伪变量`$this`。但是，如果把一个非静态的方法当作静态方法调用，那么这个`$this`是没有定义的（也就是不存在）。

#### new

如果在 new 之后跟着的是一个包含有类名的字符串，则该类的一个实例被创建。如果该类属于一个命名空间，则必须使用其完整的名称（即命名空间）。  

当把一个对象已经创建的实例赋给一个新变量时，新变量会访问同一个实例，就和用该对象赋值一样。此行为和给函数传递入实例时一样。可以用克隆给一个已创建的对象建立一个新实例。

```php
<?php
class SimpleClass
{
    // property declaration
    public $var = 'a default value';

    // method declaration
    public function displayVar() {
        echo $this->var;
    }
}

$instance = new SimpleClass();

$assigned   =  $instance;
$reference  = &$instance;

var_dump($instance);
var_dump($assigned);
var_dump($reference);
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB/1.PNG)

```php
<?php
class SimpleClass
{
    // property declaration
    public $var = 'a default value';

    // method declaration
    public function displayVar() {
        echo $this->var;
    }
}

$instance = new SimpleClass();

$assigned   =  $instance;
$reference  = &$instance;

$instance->var = '$assigned will have this value';

var_dump($instance);
var_dump($assigned);
var_dump($reference);
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB/2.PNG)

```php
<?php
class SimpleClass
{
    // property declaration
    public $var = 'a default value';

    // method declaration
    public function displayVar() {
        echo $this->var;
    }
}

$instance = new SimpleClass();

$assigned   =  $instance;
$reference  = &$instance;

$instance->var = '$assigned will have this value';

$instance = null; // $instance and $reference become null

var_dump($instance);
var_dump($assigned);
var_dump($reference);

```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB/3.PNG)

## 2、引用

引用**不是指针**。

任何其它表达式都不能通过引用传递，结果未定义。

### 引用可以做的第一件事

PHP 的引用允许用两个变量来指向同一个内容。

自 PHP 5 起，[new](http://php.net/manual/zh/language.oop5.basic.php#language.oop5.basic.new) 自动返回引用，因此在此使用 *=&* 已经过时了并且会产生 E_STRICT 级别的消息。

即不要采用如下方式：

```Php
$bar =& new fooclass();
```

不用 *&* 运算符导致对象生成了一个拷贝。如果在类中用 *$this*，它将作用于该类当前的实例。没有用 *&* 的赋值将拷贝这个实例（例如对象）并且 *$this* 将作用于这个拷贝上，这并不总是想要的结果。**由于性能和内存消耗的问题，通常只想工作在一个实例上面**。

如果在一个函数内部给一个声明为 *global* 的变量赋于一个引用，该引用只在函数内部可见。

如果在 [foreach](http://php.net/manual/zh/control-structures.foreach.php) 语句中给一个具有引用的变量赋值，被引用的对象也被改变：

```Php
<?php
$ref = 0;
$row =& $ref;
foreach (array(1, 2, 3) as $row) {
    // do something
}
echo $ref; // 3 - last element of the iterated array
?>
```

### 引用可以做的第二件事

用引用传递变量：

```php
<?php
function foo(&$var)
{
    $var++;
}

$a=5;
foo($a);
?>
```

可以将一个变量通过引用传递给函数，这样该函数就可以修改其参数的值。

将使 $a 变成 6。这是因为在 foo 函数中变量 $var 指向了和 $a 指向的同一个内容。

注意在函数调用时没有引用符号——只有函数定义中有。光是函数定义就足够使参数通过引用来正确传递了。在最近版本的 PHP 中如果把 & 用在 *foo(&$a);* 中会得到一条警告说“Call-time pass-by-reference”已经过时了。

### 引用可以做的第三件事

引用做的第三件事是[引用返回](http://php.net/manual/zh/language.references.return.php)。

引用返回用在当想用函数找到引用应该被绑定在哪一个变量上面时。*不要*用返回引用来增加性能，引擎足够聪明来自己进行优化。仅在有合理的技术原因时才返回引用！要返回引用，使用此语法：

```php
<?php
class foo {
    public $value = 42;

    public function &getValue() {
        return $this->value;
    }
}

$obj = new foo;
$myValue = &$obj->getValue(); // $myValue is a reference to $obj->value, which is 42.
$obj->value = 2;
echo $myValue;                // prints the new value of $obj->value, i.e. 2.
?>
```

和参数传递不同，这里必须在两个地方都用 *&* 符号——指出返回的是一个引用，而不是通常的一个拷贝，同样也指出 $myValue 是作为引用的绑定，而不是通常的赋值。

### 取消引用

当 unset 一个引用，只是断开了变量名和变量内容之间的绑定。这并不意味着变量内容被销毁了。例如：

```php
<?php
$a = 1;
$b =& $a;
unset($a);
?>
```

**不会 unset $b**，只是 $a。

### $this

在一个对象的方法中，$this 永远是调用它的对象的引用。

