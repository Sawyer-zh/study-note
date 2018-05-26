# 大话PHP设计模式

## 1、PHP面向对象高级特性

### PHP魔术方法的使用

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/1.png)

## 2、3种基本设计模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/2.png)

### 工厂模式

Factory.php文件里面的工厂类：

```Php
<?php
namespace IMooc;

class Factory {
	static function createDatabase() {
		$db = new Database;
		return $db;
	}
}
```

使用这个工厂类来创建对象：

```php
$db = IMooc\Factory::createDatabase();
```

使用工厂模式来创建对象的好处是，如果类的名字改了，例如`Database`换了名字，那么，我只需要修改工厂类里面的代码即可。但是，如果我们不去使用工厂模式，而是通过new的方式直接去创建一个对象，那么，每个使用new的地方都是需要修改代码的。

### 单例模式

```php
<?php
namespace IMooc;

class Database {
	static protected $db = null;
	private function __construct() {

	}

	static function getInstance() {
		if (self::$db) {
            return self::$db;
		}
		else {
            self::$db = new self();
            return self::$db;
		}
	}
}
```

使用：

```php
<?php
define('BASEDIR', __DIR__);

include BASEDIR . '/IMooc/Loader.php';
spl_autoload_register('\\IMooc\\Loader::autoload');

$db = IMooc\Database::getInstance();
```

### 注册模式

将一个对象注册到全局树上面（实际上就是把这个对象保存在一个数组里面，需要的时候就从这个数组里面去获取它即可），这样这个对象就可以在任何地方直接去访问。

 ```php
<?php
namespace IMooc;

class Register {
	protected static $objects;
	static function set($alias, $object) {
		self::$objects[$alias] = $objects;
	}

	static function _unset($alias) {
        unset(self::$objects[$alias]);
	}

	static function get($name) {
		return self::$objects[$name];
	}
}
 ```

使用：

```php
$db = IMooc\Register::get('db1');
```

## 3、适配器模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/3.png)

Database.php文件：

```php
<?php
namespace IMooc;

interface IDatabase {
	function connect($host, $user, $passwd, $dbname);
	function query($sql);
	function close();
}
```

MySQL.php文件：

```Php
<?php
namespace IMooc\Database;

use IMooc\Idatabase;

class MySQL implements IDatabase {
	protected $conn = null;

	function connect($host, $user, $passwd, $dbname) {
		$conn = mysql_connect($host, $user, $passwd);
		mysql_select_db($dbname, $conn);

		$this->conn = $conn;
	}

	function query($sql) {
		$res = mysql_query($sql, $this->conn);

		return $res;
	}

	function close() {
		mysql_close($this->conn);
	}
}
```

MySQLi.php：

```php
<?php
namespace IMooc\Database;

use IMooc\IDatabase;

class MySQLi implements IDatabase {
	protected $conn = null;

	function connect($host, $user, $passwd, $dbname) {
		$conn = mysqli_connect($host, $user, $passwd, $dbname);
		$this->conn = $conn;
	}

	function query($sql) {
		return mysqli_query($this->conn, $sql);
	}

	function close() {
		mysqli_close($this->conn);
	}
}
```

PDO.php：

```Php
<?php
namespace IMooc\Database;

use IMooc\IDatabase;

class POD implements IDatabase {
	protected $conn = null;

	function connect($host, $user, $passwd, $dbname) {
		$conn = new \PDO("mysql:host=$host;daname=$daname", $user, $passwd);
		$this->conn = $conn;
	}

	function query($sql) {
		return $this->conn->query($sql);
	}

	function close() {
		unset($this->conn);
	}
}
```

## 4、策略模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/4.png)

一般来说，针对男性女性用户需要跳转到不同的商品类目，需要使用if - else。如果这时候增加了针对新的用户的不同的广告，此时需要在代码里面继续添加 if - else 。

举个例子：

```php
class Page {
	function index() {
		if (isset($_GET['female'])) {
            // 女性
		}
		else {
            // 男性
		}
	}
}
```

此时如果又多了一种选择，那么需要继续增加 if - else：

```php
class Page {
	function index() {
		if (isset($_GET['female'])) {
            // 女性
		}
		else if (// condition) {

		}
		else {
            // 男性
		}
	}
}
```

使用策略模式可以避免这个问题。

UserStrategy.php：

```php
<?php
namespace IMooc;

interface UserStrategy {
	function showAd();
	function showCategory();
}
```

FemaleUserStrategy.php：

```php
<?php
namespace IMooc;

class FemaleUserStrategy implements UserStrategy {
	function showAd() {
		echo "2014新款";
	}

	function showCategory() {
		echo "女装";
	}
}
```

MaleUserStrategy.php：

```php
<?php

namespace IMooc;

class MaleUserStrategy implements UserStrategy {
	function showAd() {
		echo "iphone6";
	}

	function showCategory() {
		echo "电子产品";
	}
}
```

策略模式具体说就是**当需求变化时, 不要修改原有代码, 而是写新的扩展**。

```php
class Page {
	protected $strategy;

	function index() {
		echo "AD:";
		$this->strategy->showAd();

		echo "<br/>";

		echo "Category";
		$this->strategy->showCategory();
	}

	function setStrategy(\IMooc\UserStrategy $strategy) {
		$this->strategy = $strategy;
	}
}

$page = new Page;

if (isset($_GET['female'])) {
	$strategy = new \IMooc\FemaleUserStrategy();
}
else {
	$strategy = new \IMooc\MaleUserStrategy();
}

$page->setStrategy($strategy);
$page->index();
```

### 策略模式的控制反转

使用策略模式可以实现Ioc，依赖倒置、控制反转。

如果不使用策略模式，那么在控制器里面的方法里面是需要直接体现出对`FemaleUserStrategy`和`MaleUserStrategy`的依赖关系。而使用策略模式，那么在**代码执行的过程中**，才会去将Page和FemaleUserStrategy、MaleUserStrategy的关系进行一个绑定。最后执行的逻辑也是我们想要的结果。这在面向对象里面是经常用到的，因为在面向对象里面，经常需要去解耦。

## 5、数据对象映射模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/5.png)

User.php：

```php
<?php
namespace IMooc;

class User {
	public $id;
	public $name;
	public $mobile;
	public $regtime;

	protected $db = null;

	function __construct($id) {
        $this->db = new \IMooc\Database\MySQLi();
        $this->db->connect('127.0.0.1', 'root', '', 'test');       
        $res = $this->db->query("select * from user where id = $id");

        $data = $res->fetch_assoc();

        $this->id = $data['id'];
        $this->name = $data['name'];
        $this->mobile = $data['mobile'];
        $this->regtime = $data['regtime'];
	}

	function __destruct() {
        // update操作
	}
}
```

使用方法：

```php
$user = new IMooc\User(1);
$user->mobile = '111111';
$user->name = 'test';
$user->regtime = time();
```

## 6、观察者模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/6.png)

不使用观察者模式：

```php
class Event {
	function trigger() {
		echo "Event\n";

		// update
		echo "逻辑1\n";
		echo "逻辑2\n";
		echo "逻辑3\n";
	}
}

$event = new Event();
$event->trigger();
```

此时，如果update操作特别多，那么就会导致这部分代码量很大。到后面会非常难以维护。

另外一个问题就是事件和事件发生后的更新操作都是耦合在一起的。实际上事件和这3个逻辑本应该是不同的业务。

使用观察者模式可以很好的解决这个问题。

### 步骤

#### 创建一个事件生成器

EventGenerator.php：

```php
<?php

namespace IMooc;

abstract class EventGenerator {
	private $observers = array(); // 观察者对于事件发生者是不可见的，所以为私有

	function addObserver(Observer $observer) { // 增加观察者
        $this->observers[] = $observer;
	}

    function notify() {
    	// 通知观察者这个事件发生了，观察者可以进行一个更新的操作
    	// 逐个去通知所有的观察者事件发生了
    	foreach ($this->observers as $observer) {
    		$observer->update();
    	}
    }
}
```

创建一个观察者接口：

Observer.php

```php
<?php
namespace IMooc;

interface Observer {
	function update($event_info = null);
}
```

使用方式：

```php
class Event extends \IMooc\EventGenerator {
	function trigger() {
		echo "Event\n";

		$this->notify();
	}
}

class Observer1 implements \IMooc\Observer {
	function update($event_info = null) {
		echo "逻辑1\n";
	}
}

class Observer2 implements \IMooc\Observer {
	function update($event_info = null) {
		echo "逻辑2\n";
	}
}

$event = new Event();
$event->addObserver(new Observer1);
$event->addObserver(new Observer2);
$event->trigger();
```

结果：

```shell
Event
逻辑1
逻辑2
```

## 7、原型模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/7.png)

## 8、装饰器模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/8.png)

## 9、迭代器模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/9.png)

## 10、代理模式

![](http://oklbfi1yj.bkt.clouddn.com/%E5%A4%A7%E8%AF%9DPHP%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/10.png)



















