# PHP扩展开发笔记

## 1、php-config

php-config 是一个简单的命令行脚本**用于获取所安装的 PHP 配置的信息**。

在编译扩展时，如果安装有多个 PHP 版本，可以在配置时用 *--with-php-config* 选项来指定使用哪一个版本编译，该选项指定了相对应的 php-config 脚本的路径。

## 2、扩展类型

php 的扩展分为**静态编译和动态编译**两种，**静态编译就是随着PHP的源码一起编译安装**，也就是 --enable 和 --with 启用的扩展。**动态编译就是在一个已经可以使用的 PHP 环境下，使用 phpize 命令来给 php 增加扩展的方式，这种方式就是生成的 so 文件**。所以想要把扩展编译进 php 内核，就需要和 php 一起编译安装。

静态编译：

```shell
$ ./configure --prefix=/usr/local/php-7.1.16 --with-config-file-path=/usr/local/php-7.1.16/etc --with-mysqli   --with-curl --enable-fpm --enable-mbstring --with-mhash --enable-pcntl --enable-sockets --enable-zip --enable-pdo --with-pdo-mysql
```

## 3、数据结构--Function

在PHP中,函数分为俩种：
一种是zend_internal_function，这种函数是由扩展或者Zend/PHP内核提供的,用 C/C++ 编写的，可以直接执行的函数。
另外一种是zend_user_function，这种函数呢，就是我们经常在见的,用户在PHP脚本中定义的函数,这种函数最终会被ZE翻译成opcode array来执行。

PHP在编译阶段将用户自定义的函数编译为独立的opcodes,保存在EG(function_table)中,调用时重新分配新的zend_execute_data(相当于运行栈),然后执行函数的opcodes,调用完再还原到旧的zend_execute_data，即原来的函数，继续执行。

zend_function的结构中的op_array存储了该函数中所有的操作,当函数被调用时,ZE就会将这个op_array中的opline一条条顺次执行, 并将最后的返回值返回. 从VLD扩展中查看的关于函数的信息可以看出,函数的定义和执行是分开的,一个函数可以作为一个独立的运行单元而存在.

## 4、PHP CLI执行过程

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0/1.png)

基于进程的模型,**每个PHP解释器都被操作系统隔离到自己的进程中**.这种模式在Unix下很常见.

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0/2.png)

基于线程的模型,每个PHP解释器都使用线程库隔离成一个线程.该模型主要用于Windows操作系统,但也可以与大多数Unix一起使用.**这需要PHP及其扩展在ZTS模式下构建**：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0/3.png)

配置编译PHP时加参数--enable-maintainer-zts则编译出的php为**Zend线程安全**（ZTS），否则不是线程安全（NTS）。

当使用pthread（POSIX threads）扩展时，或者当web服务器为Apache2 mpm-worker或IIS使用PHP作为模块时，请考虑使用ZTS。**当使用FastCGI / FPM或Apache2 mpm-prefork时，您不需要ZTS，因为在PHP运行时使用的多进程处理**。

NTS是运行PHP的首选方式。NTS还使您更容易编写和调试扩展。

## 5、ZEND引擎编译过程

ZE是一个CISC(复杂指令处理器),正是由于它的存在,所以才能使得我们写PHP脚本时不需要考虑所在的操作系统类型是什么, 它支持170多条指令(定义在 Zend/zend_vm_opcodes.h),包括从最简单的ZEND_ECHO(echo)到复杂的 ZEND_INCLUDE_OR_EVAL(include,require),所有我们编写的PHP都会最终被处理为这些指令(op code)的序列,从而最终被执行.

从最初我们编写的PHP脚本->到最后脚本被执行->得到执行结果,这个过程,可以分为如下几个阶段：

```
* 首先,Zend Engine(ZE),调用词法分析器,将我们要执行的PHP源文件,去掉空格 ,注释,分割成一个一个的token（生成一个一个的token，所以是叫做词法分析）.
* 然后,ZE会将得到的token forward给语法分析器,生成抽象语法树.
* 然后,ZE调用zend_compile_top_stmt()函数将抽象语法树解析为一个一个的op code,opcode一般会以op array的形式存在,它是PHP执行的中间语言.
* 最后,ZE调用zend_executor来执行op array,输出结果.
```

流程图：

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0/4.png)

### TOKEN

查看一段代码的TOKEN，例如：

```php
<?php
$token =  token_get_all('<?php $str="hello world";echo $str;');
foreach ($token as $key => &$value) {
	if(is_array($value)&&(!empty($value[0]))){
		$value[0] = token_name(intval($value[0]));
	}
}
print_r($token);
```

输出：

```shell
Array
(
    [0] => Array
        (
            [0] => T_OPEN_TAG   //TOKEN名称
            [1] => <?php        //匹配到的字符
            [2] => 1            //行号
        )
    [1] => Array
        (
            [0] => T_VARIABLE
            [1] => $str
            [2] => 1
        )
    [2] => =
    [3] => Array
        (
            [0] => T_CONSTANT_ENCAPSED_STRING
            [1] => "hello world"
            [2] => 1
        )
    [4] => ;
    [5] => Array
        (
            [0] => T_ECHO
            [1] => echo
            [2] => 1
        )
    [6] => Array
        (
            [0] => T_WHITESPACE
            [1] =>  
            [2] => 1
        )
    [7] => Array
        (
            [0] => T_VARIABLE
            [1] => $str
            [2] => 1
        )
    [8] => ;
)
```

**PHP7之后的编译过程加了一层抽象语法树**,使编译过程更清晰规范,易于优化,语法规则减少,编译速度变快,编译占用内存增加。

### AST

其中`AST`是抽象语法树。

```
* https://github.com/nikic/PHP-Parser (PHP解析工具)
* https://pecl.php.net/package/ast  (扩展)
* https://dooakitestapp.herokuapp.com/phpast/webapp/ (在线)
```

### OPCEODE

**opcode是将PHP代码编译产生的Zend虚拟机可识别的指令**,php7共有173个opcode,定义在zend_vm_opcodes.h中,**这些中间代码会被Zend VM(Zend虚拟机)直接执行**.

#### opcode查看

```
* https://3v4l.org/UBstu/vld#output (在线)
* https://pecl.php.net/package/vld (扩展)
```

## 6、zend执行过程

### EG变量

executor_globals是一个全局变量,存储着许多信息(当前上下文、符号表、函数/类/常量表、堆栈等),EG宏就是用于访问executor_globals的某个成员.

## 7、扩展开发

### 编译扩展

```shell
phpize
./configure
make && make install
```

特别要注意，如果电脑上面有多个php版本，那么我们需要确保phpize和./configure命令与我们的php版本相匹配。

通过`--with-php-config`选项可以指定把扩展编译到那个版本的php：

```shell
./configure --with-php-config=/usr/local/php-7.2.3/bin/php-config
```

### 编写函数

在写完写完函数的时候，不要忘了**向PHP空间注册这个函数**，例如，实现了一个函数`say`，那么需要按照下面这样注册：

```C
const zend_function_entry say_functions[] = {
	PHP_FE(say, NULL)
	PHP_FE(confirm_say_compiled,	NULL)		/* For testing, remove later. */
	PHP_FE_END	/* Must be the last line in say_functions[] */
};
```

注意：`PHP_FE`后面**没有分号来结尾**。

PHP_FE的意思是PHP function enroll，即php函数注册。

### 创建本地变量

我们把PHP扩展中的zval结构成为变量，把PHP代码中的变量成为本地变量。

创建本地变量主要分两步，创建变量和设置为本地变量。

### PHPAPI

#### strpprintf

`strpprintf`是PHP为我们提供的字符串拼接的方法。第一个参数是最大字符数：

```c
zend_string *strpprintf(size_t max_len, const char *format, ...);
```

#### array_init_size

```c
array_init_size(return_value, zend_hash_num_elements(Z_ARRVAL_P(arr)));
```

array_init_size使用size变量初始化数组。这一行使用与键值数组一样大小来初始化数组到 `return_value` 变量里。

zend_hash_num_elements提取哈希表元素的个数（nNumOfElements属性，是数组中有效的元素，不包括被删除的元素）。这里的size只是一种优化方案。函数也可以只调用 `array_init(return_value)` ，这样随着越来越多的元素添加到数组里，PHP就会多次重置数组的大小。通过指定特定的大小，PHP会在一开始就分配正确的内存空间。

### 宏方法

我们尽量去用宏方法去操作，因为 php会升级，api也会升级，升级后结构可能会变化，所以，我们就需要用宏去访问，不关系内部实现，这个时候也更易于理解。

#### PHP_FUNCTION

定义一个 PHP 函数用的，参数就是 PHP 函数的函数名。举个例子：

```c
PHP_FUNCTION(array_change_key_case)
```

被替换成了

 ```c
void zif_array_change_key_case(zend_execute_data *execute_data, zval *return_value)
 ```

这样的一个函数定义。

#### Z_ARRVAL_P

```c
Z_ARRVAL_P
```

Z_ARRVAL_P宏从zval里面提取值到哈希表，即zend_array（HashTable和zend_array没有任何区别）。

#### ZEND_HASH_FOREACH_KEY_VAL

（哈希表的宏方法在php源码的目录`Zend/zend_hash.h`里面）

```
ZEND_HASH_FOREACH_KEY_VAL(ht, _h, _key, _val)
```

可以看出，第一个参数是整个数组结构（包括数组的一些信息，例如数组元素有效个数），而后面3个参数是数组元素`bucket`的一些成员变量。`_h`是key根据times 33计算哈希算法得到的哈希值，或者是数值索引。`_key`是存储元素的key，`_val`是存储的具体value，这里内嵌了一个zval而不是一个指针。

#### Z_TYPE_P

得到某个zval的类型。

### 对用户定义的函数的参数进行解析

例如：

```c
zend_parse_parameters(ZEND_NUM_ARGS(), "Sz", &prefix, &string)
```

- 第一个参数，参数个数。一般就使用`ZEND_NUM_ARGS()`，不需要改变。

- 第二个参数，格式化字符串。这个格式化字符串的作用就是，指定传入参数与PHP内核类型的转换关系。

  并且通过第二个参数，我们可以要求函数需要传递的参数个数。因为上面是`Sz`，有两个字符，所以一定要传递两个参数。

  `s`表示字符串，`z`表示zval，`a`表示数组。

后面还有两个参数`prefix`和`string`，这个是原来接受用户定义的函数的参数的。`prefix`与`S`对应，`string`与`z`对应。

















































