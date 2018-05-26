# Modern PHP

### 1、PHP引擎

PHP引擎是解析、解释和执行PHP代码的程序(例如Zend Engine或Facebook开发的HipHop Virtual Machine)

### 2、动态类型和静态类型

二者之间的区别在于**何时检查PHP类型**。动态类型在运行时检查类型，而静态类型在编译时检查类型

### 3、命名空间

其作用是按照一种**虚拟的层次结构**组织PHP代码，这种层次结构类似操作系统中文件系统的目录结构(即，封装和组织相关的PHP类，就像在文件系统中把相关的文件放在同一个命令一样)。现代的PHP组件和框架都放在各自全局唯一的厂商命名空间中，以免与其他厂商使用的常见类名冲突

如果没有使用命名空间，那么就会浪费对我们有用的宝贵空间

声明命名空间的代码始终应该放在`<?php`标签后的第一行

> 子命名空间之间使用一个 \ 符号分隔

PHP命名空间与操作系统的物理文件系统不同，这是一个虚拟的概念，没必要和文件系统中的目录结构完全对应。但是，为了兼容PSR-4自动加载器标准，会**把子命名空间放到文件系统的子目录中**

> 从技术层面来看，命名空间只是PHP语言中的一种记号，**PHP解释器会将其作为前缀添加到类、接口、函数和常量的名称前面**。这也是能够区分相同类名的原因

#### 为什么使用命名空间

为了在项目编写PHP组件或类时，确保这些代码和项目的第三方依赖在一起使用(因为我们的代码和其他开发者的代码使用相同的类名、接口名、函数名或常量名，如果不使用命名空间，名称会起冲突，导致PHP执行出错)

#### 命名空间的导入和别名

导入：在每个PHP文件中告诉PHP想使用那个命名空间、类、接口、函数和常量。导入后就不用输入全名了

没有导入命名空间的情况下使用命名空间：

```php
$response = new \Symfony\Component\HttpFoundation\Response('Oops', 400)
```

会发现代码很长

导入命名空间的情况，但没有使用别名：

```php
use Symfony\Component\HttpFoundation\Response;  //使用use关键字导入代码时无需在开头加上 \ 符号，因为PHP假定导入的是完全限定的命名空间

$response = new Response('Oops', 400);
```

别名：告诉PHP我要使用简单的名称引用导入的类、接口、函数或常量

导入命名空间的情况，且使用别名：

```php
use Symfony\Component\HttpFoundation\Response as Res;

$r = new Res('Oops', 400);
```

> use关键字必须出现在全局作用域中(即不能在类或函数中)，因为这个关键字在编译时使用。不过，use关键字可以在命名空间声明语句之后使用

#### 一个文件中使用多个命名空间

```php
<?php
  namespace Foo {
	//在这声明类、接口、函数和常量
}
  namespace Bar {
     //在这声明类、接口、函数和常量
 }
```

但是这样做违背了“**一个文件定义一个类**”的良好实践。如果**一个文件只使用一个命名空间，代码会更简单，而且更易于纠错**

#### 全局命名空间

如果引用类、接口、函数或常量时没有使用命名空间，PHP就会假定引用的类、接口、函数或常量在当前命名空间中。如果这个假定不正确，PHP会尝试解析类、接口、函数或常量

如果需要在命名空间中引用其他命名空间中的类、接口、函数或常量，必须使用完全限定的PHP类名(命名空间+类名)。你可以输入完全限定的PHP类名，也可以使用use关键字把代码导入当前命名空间

**如果代码没有命名空间，这些代码在全局命名空间中**

例如：

```php
<?php
  namespace My\App;
  
  class Foo {
	public function doSomething() {
      throw new \Exception();
	}
  }
```

我们在 Exception 类的名称前加上 \ 前缀，是为了告诉PHP别再当前命名空间中查找 Exception 类，因为 \My\App\Exception类并不存在。PHP此时要在全局命名空间中查找

### 4、接口

接口是两个PHP对象之间的契约，**其目的不是让一个对象依赖另一个对象的身份，而是依赖另一个对象的能力。接口把我们的代码和依赖解耦了**，而且允许我们的代码依赖任何实现了预期接口的第三方代码。我们不管第三方代码是如何实现接口的，只关心第三方代码是否实现了指定的接口

这样就可以使得那些实现了接口的类没有任何共同点，只是实现了接口

使用接口编写的代码更灵活，能委托别人实现细节

### 5、性状(trait)

性状既像类又像接口。但是**性状不是类也不是接口**

#### 性状作用

性状是类的部分实现(即常量，属性和方法)，可以混入一个或多个现有的PHP类中。性状有两个作用：表明类可以做什么(像是接口)、提供模块化实现(像是类)

性状能把模块化的实现方式注入多个无关的类中。而且性状还能促进代码重用

#### 创建性状

把 `use MyTrait;` 语句加到PHP类的定义体中即可 

```php
<?php
  trait MyTrait {
	// 这里是性状的实现，里面有一些属性和方法
}
```

> 建议一个文件只定义一个性状

#### 使用性状

```php
<?php
  class MyClass {
    use MyTrait;
    // 这里是类的实现
  }
```

然后，每个MyClass实例都能使用MyTrait性状提供的属性和方法

PHP解释器在编译时会把性状复制粘贴到类的定义体中，但是不会处理这个操作引入的不兼容问题。如果性状假定类中有特定的属性或方法(在性状中没有定义)，要确保相应的类中有对应的属性和方法

### 6、生成器

生成器是简单的迭代器，仅此而已。计算并产出后续值，**节约内存资源**

```php
<?php
function makeRange($length) {
	for ($i = 0; $i < $length; $i++) {
		yield $i;
    }
}
```

上面这个函数，一次只会为一个整数分配内存(而不是像数组那样，一开始就占用一片内存)

与标准的PHP迭代器不同，PHP生成器不要求类实现Iterator接口，从而减轻了类的负担

> PHP生成器不能满足所有迭代操作的需求，因为如果不查询，**生成器永远不知道下一个要迭代的值是什么，在生成器中无法后退或快进**。生成器是只能向前进的迭代器，**只能计算并产生下一个值**。生成器还是一次性的，无法多次迭代同一个生成器。不过，如果需要，可以重建或克隆生成器
>

#### 创建生成器

```php
<?php
function myGenerator() {
	yield 'value1';
	yield 'value2';
 	yield 'value3';
}
```

生成器就是PHP函数，只不过要在函数中一次或多次使用 yield 关键字。但是，与普通函数不同的是，**生成器不返回值，只产出值**

调用生成器函数时，PHP会返回一个属于 Generator 类的对象。这个对象可以使用 foreach() 函数迭代。每次迭代，PHP会要求 Generator  实例计算并提供下一个要迭代的值

生成器的优雅体现在，每次产出一个值之后，生成器的内部状态都会停顿；向生成器请求下一个值时，内部状态又会恢复(生成器的内部状态会一直在停顿和恢复之间切换，知道抵达函数定义体的末尾或遇到空的return;语句为止)

#### 调用和迭代生成器

```php
<?php
  foreach (myGenerator() as $yieldedValue) {
	echo $yieldedValue, PHP_EOL;
  }
```

### 7、闭包

闭包是指在创建时封装周围状态的函数。即时闭包所在的环境不存在了，闭包中封装的状态依然存在

匿名函数其实就是没有名称的函数。匿名函数可以赋值给变量，还能像其他任何PHP对象那样传递。不过**匿名函数仍是函数，因此可以调用，还可以传入参数**

匿名函数适合作为函数或方法的回调

> 理论上讲，闭包和匿名函数是不同的概念。不过，PHP将其视作相同的概念

闭包和匿名函数其实是伪装成函数的对象，它们是Closure类的实例。闭包和字符串或整数一样，也是等值类型

#### 创建闭包

```php
<?php
  $closure = function ($name) {
	return sprintf('Hello %s', $name);
};

echo $closure('Josh');
```

> 我们之所以能调用`$closure`变量，是因为这个变量的值是一个闭包，而且闭包对象实现了`__invoke()`魔术方法。只要变量名后有()，PHP就会查找并调用`__invoke()`方法

通常把PHP闭包当做函数和方法的回调使用。很多PHP函数都会用到会回调函数，例如`array_map()`。这是使用PHP匿名函数的绝佳时机

#### 附加状态

为PHP闭包附加并封装状态

在PHP中，必须手动调用闭包对象的`bindTo()`方法或者使用`use`关键字，把状态附加到PHP闭包上

##### 使用use关键字附加闭包的状态

```php
<?php
  function enclosePerson ($name) {
    return function ($doCommand) use ($name) {
		return sprintf('%s, %s', $name, $doCommand);
    }
  }

  /*
  	把字符串"Clay"封装在闭包中(注意，这里是调用了enclosePerson()函数，这个函数返回一个Closure对象<闭包对象>)。这个闭包封装了$name参数。即便返回的闭包对象跳出了具名函数enclosePerson()的作用域，它也会记住$name参数的值，因为$name变量仍在闭包对象$clay中
  */
  $clay = enclosePerson('Clay');

  //传入参数，调用闭包
  echo $clay('get ne sweet tea!');
```

> 使用use关键字可以把多个参数传入闭包，此时要像PHP函数或方法的参数一样，使用逗号分隔多个参数

### 8、Zend OPcache（字节码缓存）

#### 字节码缓存

PHP是解释型语言，PHP解释器执行PHP脚本时会解析PHP脚本代码，把PHP代码编译成一系列Zend操作码，然后执行字节码

每次请求PHP文件都是这样，会消耗很多资源，如果每次HTTP请求PHP都必须不断解析、编译和执行PHP脚本，消耗的资源更多

字节码缓存能储存预先编译好的PHP字节码。这意味着，请求PHP脚本时，PHP解释器不用每次都读取、解析和编译PHP代码。PHP解释器会从内存中读取预先编译好的字节码，然后立即执行

#### 配置Zend OPcache

```php
opcache.validate_timestamps = 1 //生产环境中(即线上)设置为“0”
opcache.revalidate_freq = 0
opcache.memory_consumption = 64
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 4000
opcache.fast_shutdown = 1
```

如果`opcache.validate_timestamps`设置为0，Zend OPcache就察觉不到PHP脚本的变化，我们必须手动清空Zend OPcache缓存的字节码，让它来发现PHP文件的变动。这个设置适合在生产环境中设置为0，但这么做在开发环境中会带来不便

### 9、内置的Web服务器

内置的服务器不应该在生产环境中使用，但对本地开发来说是极好的工具

PHP内置的是一个Web服务器。这个服务器知道怎么处理HTTP协议，能伺服静态资源和PHP文件。使用它，我们无需安装Web服务器，就能在本地编写并预览HTML

#### 启动内置的Web服务器

进入项目文档的根目录，然后执行：

```shell
php -S localhost:4000
#此命令会启动一个PHP Web服务器，地址是 localhost。这个服务器监听的端口是4000。当前工作目录是这个Web服务器的文档根目录
```

如果，我们需要在同一个局域网中的另一台设备中访问这个PHP Web服务器(例如，在手机上访问)。为此，我们可以把localhost换成0.0.0.0，让PHP Web服务器监听所有接口：

```shell
php -S 0.0.0.0:4000
```

> php -S 0.0.0.0:4000 的意思是PHP建立一个监听本机上所有IP,端口为4000的HTTP服务器.
> 所以,你在本机用下面的地址都能访问到这个PHP服务器:
> http://127.0.0.1:4000
> http://127.0.0.2:4000
> http://127.0.0.3:4000
> http://192.168.31.123:4000 (假设你的电脑在局域网的IP为192.168.31.123)
>
> 你的手机连上同一个局域网后,正常情况下,
> 通过 http://192.168.31.123:4000 就能访问你电脑上的PHP服务器.
> 如果不能,一般都是因为你的电脑的防火墙进行了拦截,
> 这时你需要配置防火墙开放4000端口,比如Linux可以这样操作:
> sudo iptables -A INPUT -p tcp -i wlan0 --dport 4000 -j ACCEPT
> 其中 wlan0 表示无线网卡.
> Windows可以在控制面板-防火墙里操作.

#### 路由器脚本

PHP内置的服务器明显遗漏了一个功能：与Apache和nginx不同，它不支持`.htaccess`文件。因此，这个服务器很难使用多数流行的PHP框架中常见的前端控制器

> 前端控制器是一个PHP文件，用于转发所有HTTP请求

PHP内置的服务器使用路由脚本弥补了这个遗漏的功能

##### 路由脚本的使用

```shell
php -S localhost:8000 router.php
#在启动PHP内置的服务器时指定这个PHP脚本文件的路径
```

#### 内置服务器确定

一次只能处理一个请求，其他请求会收到阻塞

### 10、PSR（PHP 推荐标准）

#### PSR-1：基本的代码风格

##### PHP标签

必须把PHP代码放在`<?php ?>`或`<?= ?>`标签中。不得使用其他PHP标签句法

##### 编码

所有PHP文件必须使用`UTF-8`字符集编码，而且不能有字节顺序标记

##### 目的

一个PHP文件可以定义符号（类、性状、函数和常量等），或者执行有副作用的操作（例如，生成结果或处理数据），但**不能同时做这两件事**

##### 类的名称

必须一直使用驼峰式（例如，CoffeeBean）

##### 常量的名称

必须使用大写字母。如果需要，可以使用下划线把单词分开

##### 方法的名称

必须一直使用`camelCase`这种驼峰式

#### PSR-2：严格的代码风格

##### 缩进

要求PHP代码使用四个空格缩进

> 缩进更适合使用空格，因为空格最可靠，在不同的代码编辑器中渲染的效果基本一致。而制表符的宽度各异，在不同的代码编辑器中渲染的效果也不同。为了得到最好的外观一致性，使用四个空格缩进代码
>

##### 文件和代码行

PHP文件必须使用UNIX风格的换行符，最后要有一个空行

每行代码不能超过80个字符，至少不能超过120个字符

每行末尾不能有空格

不能使用PHP关闭标签`?>`

> 最好不要写关闭标签，这样能避免意料之外的输出错误。如果加上关闭标签`?>`，而且在关闭标签后面有空行，那么这个空行会被当成输出，导致出错（例如，设定HTTP首部时）
>

##### 关键字

都应该使用小写字母，例如：`true`、`false`、`null`

##### 命名空间

每个命名空间声明语句后必须跟着一个空行

使用use关键字导入命名空间或为命名空间创建别名时，在一系列use声明语句后要加空行

```php
<?php
  namespace My\Component;

  use Symfony\Components\HttpFoundation\Request;
  use Symfony\Components\HttpFoundation\Response;

  class App
  {
      //类的定义体
  }
```

##### 类

类定义体的起始括号应该在类名之后新起一行写，结束括号必须在定义体之后新起一行写

extends和implements关键字必须和类名写在同一行

```php
<?php
  namespace My\App;

  class Administrator extends User
  {
    // 类的定义体
  }
```

##### 方法

方法定义体的起始括号要在方法名之后新起一行写，结束括号要在方法定义体之后新起一行写

**方法的参数**：其实圆括号之后没有空格，结束圆括号之前也没有空格。方法的每个参数（除了最后一个）后面有一个逗号和空格

```php
<?php
  namespace Animals;

  class StrawNeckedIbis
  {
        public function flapWings($numberOfTimes = 3; $speed = 'fast')
        {
          // 方法的定义体
        }
  }
```

##### 可见性

类中的每个属性和方法都要声明可见性（public、protected、private）

不要在类的属性前加上var关键字，**不要在私有方法的名称前加下划线**

限定符abstract或final必须放在可见性关键字之前，限定符static必须放在可见性关键字之后

```php
<?php
  namespace Animals;
  class StrawneckedIbis
  {
      //指定了可见性的静态属性
      public static $numberOfBirds = 0;
      
      //指定了可见性的方法
      public function __construct()
      {
           static::$numberOfBirds++;
      }
  }
```

##### 控制结构（if、elseif等等）

所有控制结构关键字后面都要有一个空格

控制结构关键字后面的其实括号应该和控制结构关键字写在**同一行**。控制结构关键字后面的结束括号必须单独写在一行

```php
<?php
  $gorilla = new \Animals\Gorilla;
  $ibis = new \Animals\StrawNeckedIbis;

  if ($gorilla->isAwake() === true) {
        do {
            $gorilla->beatChest();
        } while ($ibis->isAsleep() === true)；
          
        $ibis->flyAway();
  }
```

> 很多代码编辑器都能根据PSR-1和PSR-2自动格式化代码

#### PSR-4：自动加载器

自动加载器策略是指，在运行时按需查找PHP类、接口或性状，并将其载入PHP解释器

支持PSR-4自动加载器标准的PHP组件和框架，使用同一个自动加载器就能找到相关代码，然后将其载入PHP解释器

##### PSR-4自动加载器策略

PSR-4自动加载器策略依赖PHP命名空间和文件系统目录结构查找并加载PHP类、接口和性状

PSR-4的精髓是把命名空间的前缀和文件系统中的基目录对应起来

### 11、组件

现代的PHP较少使用庞大的框架，而是更多地使用具有互操作性的专门组件制定解决方案

开发新PHP应用时，可以不使用Laravel或Symfony，而是思考把哪些现有的PHP组件结合在一起解决我们的问题

#### 为什么使用组件

如果框架没有提供我们所需的功能，我们就需要自己开发。大型框架也很难集成自定义的库或第三方库，因为这些库之间没有使用相同的接口

#### 什么是组件

组件是打包的代码，用于解决PHP应用中某个具体的问题

严格来说，PHP组件是一系列相关的类、接口和性状，用于解决某个具体问题。组件中的类、接口和性状通常放在同一个命名空间中

#### 使用PHP组件

Packagist是查找PHP组件的地方，Composer是安装PHP组件的工具。Composer是PHP组件的依赖管理器，运行在命令行中。你告诉Composer需要哪些PHP组件，Composer会下载并把这些组件自动加载到你的项目中。就这么简单。Composer是依赖管理器，因此还能解析并下载组件的依赖（以及依赖的依赖，如此一直循环下去）

Composer是和Packagist紧密合作的（也就是从Packagist中获取组件的）

Composer会为项目中所有PHP组件自动生成符合PSR标准的自动加载器。Composer有效抽象了依赖管理和自动加载

### 12、过滤、验证、转义

#### 过滤

过滤输入是指，转义或删除不安全的字符。在数据到达应用层的存储层（例如redis或mysql）之前，一定要过滤输入数据

假如网站的评论表单接收HTML，默认情况下，访客可以毫无阻拦地在评论中加入恶意的`<script>`标签：

```html
<p>
  This is a help article!
</p>
<script>window.location.href='http://example.com';</script>
```

如果不过滤这个评论，恶意代码会存入数据库，然后在网站的标记中渲染。当网站的访客访问这个未过滤评论的页面时，会重定向到一个做坏事的网站

最需要过滤的输入数据类型：HTML、SQL查询和用户资料信息（例如电子邮件地址和电话号码）

##### HTML

```php
<?php
  $input = '<p><script>alert("you win the Nigerian lottery!");</script></p>';
  echo htmlentities($input, ENT_QUOTES, 'UTF-8');
```

`htmlentities()`函数会转义字符串中的HTML字符

`ENT_QUOTES`是转义单引号（如果没有这个参数，那么不会转义单引号）

第三个参数设为输入字符串的字符集

##### SQL查询

构建SQL查询不好的方式：

```php
$sql = sprintf(
    'UPDATE users SET password = "%s" WHERE id = %s',
    $_POST['password'],
    $_GET['id']
);
```

如果有人向PHP脚本发送下述HTTP请求：

```
POST /user?id=1 HTTP/1.1
Content-Length: 17
Content-Type: application/x-www-form-urlencoded

password=abc";--
```

这个请求会把每个用户的密码都设为abc，因为很多SQL数据库把`--`视作注释的开头，所以会忽略后续文本

**在SQL查询中一定不能使用未过滤的输入数据。如果需要在SQL查询中使用输入数据，要使用PDO预处理语句**

PDO预处理语句用于过滤外部数据

#### 验证

与过滤不同，验证不会从输入数据中删除信息，而是只确认输入数据是否符合预期。如果遇到无效数据，要中止数据存储操作，并显示适当的错误消息来提醒应用的用户

> 输入数据既要验证也要过滤，一次确保输入数据是安全的，而且符合预期

#### 转义输出

**把输出渲染成网页或API响应时，一定要转义输出**。这也是一种防护措施，能避免渲染恶意代码，还能防止应用的用户无意中执行恶意代码

### 13、密码

#### 绝对不能知道用户的密码

我们绝对不能知道用户的密码

#### 绝对不能通过电子邮件发送用户的密码

如果你通过电子邮件给我发送密码，我会知道三件事：你知道我的密码；你是要纯文本或能解密的格式存储了我的密码；你没有对通过互联网发送纯文本的密码感到不安

#### 使用bcrypt计算用户密码的哈希值

加密和哈希是两码事。**加密是双向算法**，加密的数据以后可以解密。而**哈希是单向算法**，哈希后的数据不能再还原成原始值，而且**相同的数据得到的哈希值始终是一样的**

哈希算法有很多种（MD5、SHA1、bcrypt和scrypt）。有些算法的速度很快，用于数据完整性；有些算法的速度慢，旨在提高安全性。**生成密码和存储密码需要使用速度慢、安全性高的算法**

**最安全的哈希算法是bcrypt**

**密码的哈希值要存储在VARCHAR(255)类型的数据库列中**（这样便于以后存储比现在的bcrypt算法得到的哈希值更长的密码）

### 14、日期、时间和时区

应该使用PHP的DateTime、DateInterval和DateTimeZone类来处理

#### 设置默认时区

可以在配置文件中设置，也可以在代码运行时设置

### 15、数据库

#### PDO扩展

PDO（PHP Data Objects的简称，意思是“PHP数据对象”）是一系列PHP类，抽象了不同数据库的具体实现，只通过一个用户界面就能与多种不同的SQL数据库通信。**不管使用哪种数据库系统，使用一个接口就能编写和执行数据库查询**

> 虽然PDO扩展为不同数据库提供了同一接口，但是我们仍然必须自己编写SQL语句。这是PDO的劣势所在。各种数据库都会提供专属的特性，而这些特性通常需要独特的SQL句法。所以建议使用PDO时编写符合ANSI/ISO标准的SQL语句，这样如果更换数据库系统，SQL语句不会失效

#### 事务

PDO扩展还支持事务

### 16、多字节字符串

PHP假设字符串中的每个字符都是8位字符，占一个字节的内存。可是这种假设很天真，一旦处理非英文字符就失效了

这里的多字节字符是指，不在传统的128个ASCII字符集中的字符（例如中文字符）

PHP中处理字符串的函数默认假设所有字符串都只使用8位字符，如果使用这些PHP原生的字符串函数处理包含多字节字符的Unicode字符串，会得到意料之外的错误结果

为了避免处理多字节字符串时出错，可以安装mbstring扩展。这个扩展提供了知道如何处理多字节字符串的函数，能代替大多数PHP原生的处理字符串的函数。例如，知道如何处理多字节字符串的`mb_strlen()`

函数用于代替PHP原生的`strlen()`函数

#### 字符编码

字符编码是打包Unicode数据的方式以便把数据存储在内存中，或者通过线缆在服务器和客户端之间传输

处理多字节字符串时，要记住：

> 1、一定要知道数据的字符编码
>
> 2、使用UTF-8字符编码存储数据
>
> 3、使用UTF-8字符编码输出数据

建议在HTML文档的头部加入下述meta标签：

```html
<meta charset="UTF-8"/>
```

### 17、流

作用是使用统一的方式处理文件、网络和数据压缩等共用同一套函数和用法的操作。流的作用是在出发地和目的地之间传输数据。出发地和目的地可以是文件、命令行进程、网络连接、ZIP或TAR压缩文件、临时内存、标准输入或输出，或者是通过PHP流封装协议实现的任何其他资源

> 可以把流理解为管道，相当于把水从一个地方引到另一个地方。在水从出发地留到目的地的过程中，我们可以过滤水，可以改变水质，可以添加谁，也可以排出水（水代指数据）

#### 流封装协议

例如，我们可以读写文件系统，可以通过HTTP、HTTPS或SSH（安全的shell）与远程Web服务器通信，还可以打开并读写ZIP、RAR或PHAR压缩文件。这些通信方式都包含下述相同的过程：

> 1、开始通信
>
> 2、读取数据
>
> 3、写入数据
>
> 4、结束通信

每个流都有一个协议和一个目标。指定协议和目标的方法是使用流标识符：

```php
<scheme>://<target>
```

其中，`<scheme>`是流的封装协议，`<target>`是流的数据源

举个例子，使用HTTP封装协议与Flickr API通信

```php
<?php
  $json = file_get_contents(
      'http://api.flickr.com/services/feeds/photos_public.gne?format=json'
  );
```

不要误以为这是一个普通的网页URL，file_get_contents()函数的字符串参数其实是一个流标识符。http协议会让PHP使用HTTP流封装协议。在这个参数中，http之后就是流的目标。流的目标之所以看起来像是普通的网页URL，是因为HTTP流封装协议就是这样规定的。其他流封装协议可能不是这样

> 普通的URL其实是PHP流封装协议标识符的伪装

##### file://流封装协议

我们使用`file_get_contents()`、`fopen()`、`fwrite()`和`fclose()`函数读写文件系统

因为**PHP默认使用的流封装协议是`file://`**，所以我们很少认为这些函数使用的是PHP流

**隐式使用file://封装协议**：

```php
<?php
  $handle = fopen('/etc/hosts', 'rb');
  while () {
      echo fgets($handle);
  }
  fclose($handle);
```

**显示使用file://封装协议**：

```php
<?php
  $handle = fopen('file:///etc/hosts', 'rb');
  while () {
      echo fgets($handle);
  }
  fclose($handle);
```

##### php://流封装协议

这个流封装协议的作用是与PHP脚本的标准输入、标准输出和标准错误文件描述符通信

**php://stdin**

这是个只读PHP流，其中的数据来自标准输入。例如，PHP脚本可以使用这个流接收命令行传入脚本的信息

**php://stdout**

把数据写入当前的输出缓冲区

**php://memory**

从系统内存中读取数据，或者把数据写入系统内存

缺点：内存有限

**php://temp**

和 php:memory 类似，不过没有可用内存时，PHP会把数据写入临时文件

##### 其他流封装协议

PHP中的`fopen()`、`fgets()`、`fputs()`和`fclose()`等文件系统函数不仅仅能用来处理文件系统中的文件，其实这些文件系统函数能在所有支持这些函数的流封装协议中使用

#### 流上下文

流上下文使用`stream_context_create()`函数创建，这个函数返回的上下文对象可以传入大多数文件系统和流函数

例如，可以使用`file_get_contents()`函数发送HTTP POST请求：

```php
<?php
  $requestBody = '{"username":"josh"}';
  $context = stream_context_create(array(
      'http' => array(
          'method' => 'POST',
          'header' => "Content-Type: application/json;charset=utf-8;\r\n" .
                       "Content-Length: " .mb_strlen($requestBody),
          'content' => $requestBody
      )
  ));
  $response = file_get_contents('https://my-api.com/users', false, $context);
```

流上下文是个关联数组，最外层是流封装协议的名称

### 18、错误和异常

错误和异常之间很相似，容易混淆。错误和异常都表明出问题了，都会提供错误消息，而且都有错误类型。然而，错误出现的时间比异常早。错误会导致程序脚本停止执行，如果可能，错误会委托给全局错误处理程序处理。有些错误是无法恢复的

**我们基本上只需处理异常，不管错误**

异常是PHP的错误处理系统向面向对象演进后得到的产物

**异常要先实例化，然后抛出，最后再捕获**

异常是预期并处理问题更为灵活的方式，可以就地处理，无需停止执行脚本。异常进可攻退可守。我们必须使用`try {} catch{}`代码块预测第三方厂商的代码可能抛出的异常

#### 异常

异常是`Exception`类的对象。`Exception`对象使用`new`关键字实例化。`Exception`对象有两个主要属性：一个是消息，另一个是数字代码（消息用于描述出现的问题；数字代码是可选的，用于为指定的异常提供上下文）

```php
<?php
  $exception = new Exception('Danger, will Robinson!', 100);
```

可以使用公开的实例方法`getCode()`和`getMessage()`获取`Exception`对象的这两个属性：

```php
<?php
  $code = $exception->getCode(); //100
  $message = $exception->getMessage(); // 'Danger...'
```

##### 抛出异常

实例化时可以把异常赋值给变量，不过一定要把异常抛出。抛出异常后代码会立即停止，后续的PHP代码都不会运行。抛出异常的方式是使用`throw`关键字，后面跟着要抛出的`Exception`实例：

```php
<?php
  throw new Exception('Something went wrong. Time for lunch!');
```

上面的代码会获得一个致命错误和一个异常

要避免上面这个致命错误，可以使用try catch捕获掉

##### 捕获异常

未捕获的异常会导致PHP应用终止运行，显示致命错误信息

拦截并处理潜在异常的方式是，把可能抛出异常的代码放在`try/catch`块中。`catch`块会捕获这个异常，然后显示一个友好的错误消息，而不是丑陋的堆栈跟踪

可以使用多个`catch`块拦截多种异常。如果要使用不同的方式处理抛出的不同异常类型，可以这么做。还可以使用一个`finally`块，在捕获任何类型的异常之后运行一段代码：

```php
<?php
  try {
      throw new Exception('Not a PDO exception');
      $pdo = new PDO('mysql://host=wrong_host;dbname=wrong_name');
  } catch (PDOException $e) {
      // 处理PDOException异常
      echo 'Caught PDO exception';
  } catch (Exception $e) {
      // 处理所有其他类型的异常
      echo 'Caught generic exception';
  } finally {
      // 这里的代码始终都会运行
      echo 'Always do this';
  }
```

### 19、PHP-FPM

PHP-FPM（PHP FastCGI Process Manager的简称，意思是“PHP FastCGI进程管理器”）是用于管理PHP进程池的软件，用于接收和处理来自Web服务器（例如nginx）的请求

**PHP-FPM软件会创建一个主进程（通常以操作系统中根用户的身份运行），控制何时以及如何把HTTP请求转发给一个或多个子进程处理**。PHP-FPM主进程还控制着什么时候创建（处理Web应用更多的流量）和销毁（子进程运行时间太久或不再需要了）PHP子进程。PHP-FPM进程池中的每个进程存在的时间都比单个HTTP请求长，可以处理10、50、100、500或更多的HTTP请求





























































































 