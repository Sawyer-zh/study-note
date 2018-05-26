大家应该都听说过SQL注入吧，但是，网络上的例子貌似都是一些**代码片段**，如果一个新手想要自己尝试一下SQL注入来直观的感受一下SQL注入的危害，显得很无助。我刚学习这方面知识的时候也苦苦找不到这样的代码。最近在知乎上看到很多人都提关于SQL注入的问题，于是，打算写一篇这样的博客。

直接上代码：

先创建数据表：

```mysql
CREATE TABLE IF NOT EXISTS `users`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `username` varchar(200) NOT NULL,
   `password` CHAR(32) NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE = InnoDB DEFAULT CHARSET = utf8;

INSERT INTO `users` (`id`, `username`, `password`) VALUES
(1, 'admin', MD5('admin')),
(2, 'huanghantao', MD5('huanghantao'));
```

下面是结果：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/1.png)

（密码是结果md5函数加密的）

然后是PHP脚本处理登录逻辑的代码：

```php
<?php
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "test";
 
// 创建连接
$conn = new mysqli($servername, $username, $password, $dbname);
 
// 检测连接
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
}

$username = "admin";
$password = "admin";
$sql = "SELECT * FROM users where username = '" . $username . "' AND password = '" . md5($password) . "'";
$result_obj = $conn->query($sql);

if ($result_obj->num_rows) {
	while($row = $result_obj->fetch_assoc()) {
		var_dump($row);
    }
}
```

OK，程序大概就是这样了，

现在我们执行上面这段PHP脚本：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/2.png)

嗯，符合我们的预期。（因为用户名和密码都正确，所以打印出了用户的结果）

好的，接下来开始我们的表演：

我们修改一下PHP脚本的username变量：

```php
$username = "admin' --";
$password = "密码随便写"; // 没错，你写中文也行
```

完整代码如下：

```php
<?php
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "test";
 
// 创建连接
$conn = new mysqli($servername, $username, $password, $dbname);
 
// 检测连接
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
}

$username = "admin' --";
$password = "代码随便写";
$sql = "SELECT * FROM users where username = '" . $username . "' AND password = '" . md5($password) . "'";
$result_obj = $conn->query($sql);

if ($result_obj->num_rows) {
	while($row = $result_obj->fetch_assoc()) {
		var_dump($row);
    }
}
```

我们执行代码：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/3.png)

说的是`$result_obj`变量不是一个对象，所以没有属性。好的，我们打印一下`$result_obj`看看它究竟是什么：

```php
var_dump($result_obj);
```

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/4.png)

原来返回的是一个boolean类型的变量。说明我们的SQL语句没有执行成功。这就是很坑的一个地方了，网络上的文章大多数是草草的告诉我们，在表单填写类似这样的东西就可以了：

```
admin' --
```

即加个单引号 `'`，加两个 `-`。实际上，或许你得再加点东西。

我们修改下代码：

```php
$username = "admin' -- // ";
$password = "代码随便写";
```

也就是说再加两个`/`。

好的，我们执行一下代码：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/5.png)

怎么样，就算我们不知道密码，只要我们知道这个用户的用户名，就可以登录到这个用户的页面对吧。

好的，接下来，我们再试试，即使没有用户名和密码，我们也可以得到所有用户的用户名和密码：

```php
$username = "' or 1 -- // ";
$password = "代码随便写";
```

执行后的结果：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/6.png)

很可怕对吧。

至于这两段代码中的：

```
admin' --
和
' or 1 --
```

什么意思，网上的文章已经讲烂了，就不多说了。

通过什么的操作，我们可以发现，那个单引号`'`特别捣蛋对吧，所以，我们修改下代码，对单引号进行转义：

```php
$username = "' or 1 -- // ";
$username = addslashes($username);
$password = "代码随便写";
```

执行后的结果：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/7.png)

可以看到，并没有打印出用户信息，所以这个函数阻挡了此次攻击。原因如下：

![](http://oklbfi1yj.bkt.clouddn.com/SQL%E6%B3%A8%E5%85%A5/8.png)

`or`前面的单引号被转义了。

但是，并不建议通过`addslashes`对单引号转义的方式来阻挡这种类型的SQL注入。因为**addslashes对有些特殊字符后跟上'（单引号）并不会加把这个'变成/'**。

这也是为什么很多人建议使用PDO来对数据库进行操作的原因了。

好的，文章到这里结束了，希望对大家有所帮助。文中的代码都可以复制后直接使用，希望可以减少大家踩坑的时间。

happy ending…...





