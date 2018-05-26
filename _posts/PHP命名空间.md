---
title: PHP命名空间
date: 2018-01-05 14:13:49
tags:
- PHP
---

这篇文章是关于PHP**命名空间**的一些使用方法的介绍。命名空间的概念我就不多说了，随便百度一下就好了。

直接上代码：

```php
<?php
  
    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }
    
    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }
```

很多博客可能会千篇一律的写一段这样的代码。然后再接着写：

```php
new Person();
```

然后问大家输出什么结果，其实这样很没意思，至少我是这么觉得。这样学习是很不对的。

emmm，先看一下结果吧：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/1.PNG)

现在我给大家介绍一个常量：`__NAMESPACE__`。这个常量保存了当前区域所处的命名空间。有了这个东西，就不用猜测了，一切都很明显。我们来测试一下：

```php
<?php

    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }
    
    var_dump(__NAMESPACE__);
    
    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }
```

我们执行这段代码：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/2.PNG)

说明`namespace School\Teacher;`和`namespace School\Parent;`之间的代码是属于命名空间`School\Teacher`的。OK，我们继续测试：

```php
<?php

    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }

    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }

    var_dump(__NAMESPACE__);
```

我们执行一下这段代码：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/3.PNG)

说明在`namespace School\Parent;`之后的区域都属于命名空间`School\Parent`的。

好的，到了这一步，我们就明白为什么在上面实例化`Person`类的时候，是输出`I am Parent`了吧。

但是，这两个类名都是同样的呀，PHP它是如何区分的呢？这里我还不确定，但是我猜测底层为我们加上了命名空间前缀。（先保留这个问题，等我看了源码之后，再来回答）

OK，那如果我非要在`School\Parent`命名空间中实例化`School\Teacher`命名空间中的Person呢？我们来做个测试一下：

```php
<?php

    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }

    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }

    $obj = new School\Teacher\Person();
    var_dump($obj);
```

这样会报错：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/4.PNG)

我们可以看到，它是在`School\Parent`的命名空间下去寻找命名空间`School\Teacher`中的`Person`类。那么这是不是很像我们平常接触的目录的相对路径？所以我们需要修改一下代码：

```php
$obj = new \School\Teacher\Person();
```

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/5.PNG)

那么这种写法是不是很像绝对路径？这说明了一个问题，命名空间应该是有一个`根`。 

大家会不会觉得`new`后面跟了很长的一段东西，OK，我们把它改简单一点。这时候我们可以使用`use`关键字：

```php
<?php

    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }

    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }

    use School\Teacher;
    new Person();
```

我们看看结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/6.PNG)

我们发现和上面的结果不一样对吧，这次我们实例化的是`School\Parent`命名空间中的`Person`类。很奇怪啊，上面我明明使用了：

```php
 use School\Teacher;
```

语句啊，为什么还是在`School\Parent`命名空间中搜索`Person`类呢？很多初学者如果不去仔细分析的话，很可能会有这样的疑问。

我们修改一下代码，打印一下命名空间：

```php
use School\Teacher;
var_dump(__NAMESPACE__);
new Person();
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/7.PNG)

这说明了`use School\Teacher;`并不是使后面的命名空间变为`School\Teacher`。这下知道上面为什么是实例化`School\Parent`命名空间下的`Person`类了吧。

那么`use`到底是什么作用呢？OK，其实它是起到了一个简写的作用。也就是说，我们可以用`use School\Teacher;`中最后的那个`Teacher`代替`\School\Teacher`（注意不是代替`School\Teacher`，通过我们上面的实验可以验证这一点）。

OK，知道了这个之后，我们再次修改一下代码：

```php
use School\Teacher;
var_dump(__NAMESPACE__);
new Teacher\Person();
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/8.PNG)

OK，果然是这样的对吧。

如果还是不信的话，我再次修改一下代码：

```php
use School\Teacher\Person;
var_dump(__NAMESPACE__);
new Person();
```

结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/9.PNG)

说的是不能够`use School\Teacher\Person as Person`。有人会问后面的`as Person`是什么鬼？其实`as Person`的意思是用`Person`来代替`\School\Teacher\Person`。但是我们在代码里面没有使用`as Person`呀？我个人觉得是因为它默认会取`use School\Teacher\Person`中的最后一个（即Person）来`as`。我们可以测试一下：

```php
use School\Teacher\Person as Person;
var_dump(__NAMESPACE__);
new Person();
```

得到如下结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/10.PNG)

一样的报错对吧。

OK，既然它说不能够`as Person`的话，那我就换一个呗：

```php
use School\Teacher\Person as Teacher;
var_dump(__NAMESPACE__);
new Teacher();
```

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/11.PNG)

这样的结果可以证明我说的`use`的作用了吧。

接下来简单演示一下多个文件中如何使用`use`（用过Laravel、ThinkPHP框架的人一定知道），我发现很多博客都没有讲这一点，导致一些新手容易产生一个幻觉。

首先，我建立两个文件：`Parent.php`和`Teacher.php`。

我们还是使用上面的代码但是，把这两个类分别写在这两个文件里面：

```php
<?php
    // Teacher.php
    namespace School\Teacher;
    class Person {
        function __construct() {
            echo 'I am Teacher';
        }
    }
```

```php
<?php
    // Parent.php
    namespace School\Parent;
    class Person {
        function __construct() {
            echo 'I am Parent';
        }
    }
```

然后我们在`test.php`文件里面分别去实例化这两个类：

```php
<?php

    use School\Teacher\Person as Teacher;
    new Teacher();
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4/12.PNG)

咦，我们使用别人写的框架的时候，不是可以做类似跨文件的操作吗？（反正当初菜鸡博主尝试过，以为是可以的。然而框架欺骗了我的感情）

其实小伙伴们被框架给迷惑了，PHP至少现在还不支持这种骚操作。（我也觉的不要这样子，代码很容易混乱的）

框架的真实情况可能是这样的，把这两个类文件在一个入口文件里面引入，然后才能进行这样的操作。例如，菜鸡博主最近在开发一个框架，就有类似的代码：

```php
<?php

	/*
	应用（项目）的入口文件index.php
	*/

    include 'HHTCore/Common/bootstrap.php';
    include 'HHTCore/Controller/Controller.php'; // 注意这个地方引入了一个文件

    $app = isset($_GET['app']) ? $_GET['app'] : 'Home'; // 默认的应用是Home
    $controller = isset($_GET['c']) ? $_GET['c'] : 'Index'; // 默认的控制器是Home目录下的IndexController

    /* some code */

    if (is_file($file)) {
    	include $file; // 这里又引入了一个文件
    	$controller = '\APP\Home\Controller\\' . $controller;

    	$appClassObject = new $controller();
    	$appClassObject->$action();
    }
    else {
    	echo 'error';
    }
```

然后，又有两个文件：

```php
<?php
    // Controller.php
    namespace HHTCore\Controller;

    class Controller {
    	protected function render($view = '', $data = []) {
            var_dump(dirname(__FILE__));
    	}
    }
```

```php
<?php
    // IndexController.php
    namespace APP\Home\Controller;

    use HHTCore\Controller\Controller;

    class IndexController extends Controller {
    	public function index () {
    		$this->render();
    	}
    }
```

看起来我在`IndexController.php`里面使用了`use HHTCore\Controller\Controller;`之后，就可以直接使用类`Controller`了。然而不是的，我们在上面的`Teacher.php`和`Parent.php`做过实验。真实情况是，我们每次访问我们的Web项目，都是要从那个入口文件`index.php`里面去访问的，所以每次都会去引入那两个文件（其中后面的那个文件是根据URL动态变化的），因此，就会出现我们上面的错觉。

这篇博客就和大家分享到这里吧。

我希望我的每一篇博客都用心去写。

happy ending......

