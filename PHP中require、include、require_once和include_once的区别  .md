# PHP中require、include、require_once和include_once的区别

require 和 include 几乎完全一样。

## 出错时候的处理方式

require 在出错时产生 E_COMPILE_ERROR 级别的错误。换句话说将**导致脚本中止**，而 include 只产生警告（E_WARNING），脚本会继续运行。

例如：

```php
<?php
include 'index.html'; // index.html不存在

var_dump('expression');
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E4%B8%ADrequire%E3%80%81include%E3%80%81require_once%E5%92%8Cinclude_once%E7%9A%84%E5%8C%BA%E5%88%AB/1.png)

```php
<?php
require 'index.html';

var_dump('expression');
```

结果：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E4%B8%ADrequire%E3%80%81include%E3%80%81require_once%E5%92%8Cinclude_once%E7%9A%84%E5%8C%BA%E5%88%AB/2.png)

## include与require使用方法

require 的使用方法如 `require("./inc.php");`通常放在 PHP 程序的最前面，PHP 程序在执行前，就会先读入 require 所指定引入的文件，使它变成 PHP 程式网页的一部份。 include 使用方法如 `include("./inc/.php");`一般是放在流程控制的处理区段中。PHP 程式网页在读到 include 的文件时，才将它读进来。这种方式，可以把程式执行时的流程简单化。

## 使用require还是require_once呢？

使用require（而不是require_once）可以提高应用程序的性能。

由于在导入PHP脚本时将进行大量的操作状态(stat)调用，因此require要快于require_once。如果你请求的文件位于目录/var/shared/htdocs/myapp/ models/MyModels/ClassA.php下，则操作系统会在到达ClassA.php之前的某个目录中运行一次stat调用。在这个例子中，共进行了6次stat调用。当然， require也会发起stat调用，但是次数较少。越少的函数调用，代码的运行速度越快。

但是，一般情况下，也不太需要去考虑究竟require还是require_once，除非这已经严重影响到你程序的性能了。



