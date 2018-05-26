# PHP fastcgi_finish_request 方法

## PHP运行模式

### CGI 通用网关接口 / Common Gateway Interface

CGI已经是比较老的模式了，这几年都很少用了。

介绍：每有一个用户请求，都会先要创建CGI的子进程，然后处理请求，处理完后结束这个子进程，这就是Fork-And-Execute模式。

当用户请求数量非常多时，会大量挤占系统的资源如内存，CPU时间等。

缺点：在高访问需求的情况下，CGI的进程Fork就会成为很大的服务器负担。

### FastCGI（常驻型CGI / Long-Live CGI）

使用的比较多。

介绍：FastCGI是CGI的升级版本，FastCGI像是一个**常驻 (long-live)型**的 CGI。

它可以一直执行着，只要激活后，不会每次都要花费时间去 Fork 一次。

FastCGI是一个可伸缩地、高速地**在HTTP server和动态脚本语言间通信的接口**。

原理：

1.Web Server启动时载入FastCGI进程管理器（PHP-FPM）；

2.FastCGI进程管理器初始化启动多个CGI解释器进程并等待来自Web Server的连接；

3.当客户端请求到达Web Server时，**FastCGI进程管理器选择并连接到一个CGI解释器**；

4.**Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi**；

5.FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。

当FastCGI子进程关闭连接（不是FastCGI子进程被关闭，而是连接被关闭）时，请求便处理完成。

FastCGI子进程接着等待并处理来自FastCGI进程管理器的下一个连接。

## fastcgi_finish_request

作用：冲刷(flush)所有响应的数据给客户端。

个人理解：在调用方法的时候，会发送响应，关闭连接，但是不会结束PHP的运行。

个人理解：在调用方法的时候，会发送响应，关闭连接，但是不会结束PHP的运行。

不理解的可以直接运行如下代码：

```php
//代码：
echo date('Y-m-d H:i:s', time())."\r\n"; //会输出

fastcgi_finish_request();

set_time_limit(0);  //避免超时报错

ini_set('memory_limit', '-1');  //避免内存不足

sleep(5);

$time = date('Y-m-d H:i:s', time())."\r\n";

echo $time; //不会输出

file_put_contents('test.txt', $time, FILE_APPEND);
```

执行这段函数后你会发现，可以实现异步操作，提高响应速度。

可以使用fastcgi_finish_request()函数集成队列，可以把消息异步发送到队列。

因为这个函数只在FastCGI模式下存在，考虑可移植性可以加上以下代码：

```php
if (!function_exists("fastcgi_finish_request")) {
      function fastcgi_finish_request()  {
      }
}
```







