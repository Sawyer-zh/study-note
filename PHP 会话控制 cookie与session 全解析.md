# PHP 会话控制 cookie与session 全解析

1、一般情况下，Cookie通过**HTTP headers**从服务端返回到客户端。多数web程序都支持Cookie的操作，**因为Cookie是存在于HTTP的标头之中，所以必须在其他信息输出以前进行设置**，类似于header函数的使用限制

2、**任何从浏览器发回的Cookie，PHP都会自动的将他存储在 ` $_COOKIE`的全局变量之中**，因此我们可以通过` $_COOKIE[‘key’] `的形式来读取某个Cookie值

**`setcookie()` 函数向客户端发送一个 HTTP cookie**。cookie 是由服务器发送到浏览器的变量(达到了设置cookie的目的)

3、**设置cookie的方法**除了`setcookie()`函数以外，还可以用header()函数

因为Cookie是通过HTTP标头进行设置的，所以也可以直接使用header方法进行设置：

```php
header("Set-Cookie:cookie_name=value");
```

4、Cookie 的有效路径

cookie 中的路径用来控制设置的cookie在哪个路径下有效，默认为 `/` ，在所有路径下都有

```php
// 使设置的这个cookie在/path以及子路径/path/abc下都有效，但是在根目录下就读取不到这个cookie值。
setcookie('CookieName', 'CookieValue', time() + 3600, '/path');
```

5、用户信息既可以存储在session中，也可以存储在cookie中，他们之间的差别在于**session可以方便的存取多种数据类型**(**session会自动的对要设置的值进行encode与decode**，因此session可以**支持任意数据类型**，包括数据与对象等)，而cookie只支持字符串类型，同时对于一些安全性比较高的数据，cookie需要进行格式化与加密存储，而session存储在服务端则安全性较高

6、通过一个`session_id`进行用户识别，**PHP默认情况下`session_id`是通过cookie来保存的**，因此**从某种程度上来说，seesion依赖于cookie。**但这不是绝对的，`session_id`也可以通过参数来实现，**只要能将`session_id`传递到服务端进行识别的机制都可以使用session**

7、删除 session

如果要删除所有的session，可以使用 **session_destroy()** 函数，但是 **session_id 仍然存在**，只有当下次再访问的时候，session才为空，因此**如果需要立即销毁`$_SESSION`，可以使用unset函数**

如果需要销毁session_id，通常在用户退出的时候可能会用到，则还需要显式的调用setcookie方法删除session_id的cookie值

































