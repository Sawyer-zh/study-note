# hht-php-frame框架

## 可以使用定义的常量的文件

都放在了`HHTCore/Common/bootstrap.php`文件中了。

## 自动加载项目Model文件的函数

`HHTCore/Common/function.php`文件中的`autoLoadModel`函数

## 使用框架核心代码中的Model里面的方法

```php
<?php
    namespace APP\Home\Controller;

    use HHTCore\Controller\Controller;
    use APP\Home\Model\UsersModel;

    class IndexController extends Controller {
    	public function index () {
    		$users = new UsersModel();
    		$res = $users->get()->find(1);
    		var_dump($res);
    	}
    }
```

注意，在使用了：

```php
$users = new UsersModel();
```

之后，需要使用：

```php
$users->get()
```

才能使用框架核心代码自带的方法。

## 输出模板页面的函数

`HHTCore/Controller/Controller.php`文件中的`render`。

## 接入Smarty模板引擎

开始是想要让框架的核心类Controller继承Smarty类，但是这两个类千差万别，所以不可能是继承关系。所以就只让Controller类获得一个Smarty对象即可。这样，就可以使用Smarty类中的方法了。

同时，因为框架的核心类Controller是不需要被实例化的，所以把他修改为抽象的。