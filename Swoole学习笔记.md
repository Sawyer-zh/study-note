# Swoole学习笔记

1、实现进程间通信的方法有多种，例如**共享内存**。共享内存是操作系统中一个比较特殊的内存，它并不依赖于任何进程(不需要依赖进程而存在)，不属于任何进程

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/1.PNG)

共享内存不属于任何进程、在共享内存中分配的存储空间可以被任何进程访问、即使进程关闭，共享内存仍然可以继续保留

在linux中可以通过命令：`ipcs -m`来查看共享内存的分配情况

查看到的`shmid`是这片共享内存的索引，`perms`是访问权限

2、Swoole的结构：

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/2.PNG)

其中橙色代表进程，蓝色的代表线程

Master是处理Swoole的事件的核心驱动的，它拥有若干个线程，Swoole所有的对事件的监听都会在这些线程当中实现，比如来自客户端的连接、本地通讯用的管道、异步操作的文件

worker进程是swoole的主逻辑进程，用于处理来自客户端的请求

task进程是swoole提供的异步工作进程，主要用于处理一些耗时比较长的同步任务

在swoole当中，进程与进程之间的通信是**基于管道**来实现的

它和fpm是不同的，fpm是来了一次新的请求就创建一个进程去处理这个请求，这样，这个系统在很大的程度上的消耗是用在了创建和销毁进程，导致整个系统的相应效率并不是非常的高

3、

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/3.PNG)

task传递只能传递一个字符串，所以，如果是要传递数组的话，要用`json_encode()`对数组进行编码

4、Task常见问题

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/4.PNG)

5、启动server.php后，当前进程fork出Master进程，然后推出

6、server端好了，那么就会需要client端来连接，**swoole里面client分为同步(sync)和异步**

7、系统服务中，端口不是独立存在的，它是依附于进程的。某个进程开启，那么它对应的端口就开启了，进程关闭，则该端口也就关闭了。下次若某个进程再次开启，则相应的端口也再次开启。所以，不要纯粹的认为你要去关闭掉某个端口了。但是，禁用某个端口是可行的。因为，linux里面端口的活动与进程是紧密相连的，如果想要关闭某个端口，那么只要kill掉与它对应的进程就可以了

8、可以使用命令：`netstat -an | grep 端口`来查看端口是否已经被打开处于Listening状态

9、调用 `$server->close()` 方法可以强制关闭某个客户端连接

​      **客户端可能会主动断开连接**，此时会触发onClose事件回调

10、一个简单的TCP服务器的创建

```php
<?php
//命名为server.php
  
//创建Server对象，监听 127.0.0.1:9501端口
$serv = new swoole_server("127.0.0.1", 9501); 

//监听连接进入事件
$serv->on('connect', function ($serv, $fd) {  
    echo "Client: Connect.\n";
});

//监听数据接收事件
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, "Server: ".$data);
});

//监听连接关闭事件
$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//启动服务器
$serv->start(); 
```

执行程序：`php server.php`

**然后使用telnet来连接服务器：`telnet 127.0.0.1 9501`**

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/6.PNG)

连接完了之后，我们返回运行`server.php`的那个终端，服务器的终端会显示：`Client：Connect`

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/5.PNG)

也就是说执行了这段代码：

```php
$serv->on('connect', function ($serv, $fd) {  
    echo "Client: Connect.\n";
});
```

此时，我们在客户端的那个终端输入数据：

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/7.PNG)

也就是说执行了这段代码：

```php
//监听数据接收事件
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, "Server: ".$data);
});
```

而传入的`$fd`是用来标识是哪一个客户端(因为可能有很多个客户端来向这个服务端发起连接，所以要标识起来)发来的数据，这样，我们也可以利用`$fd`来从服务端发送数据回去那个客户端，这样就实现了通信

`$data`是从客户端接收到的数据

**注意：在启动服务器之前，必须得先使用`receive`或`packet`回调函数，否则会报错**：

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/16.PNG)

11、一般情况下测试端口是否监听都是使用的telnet命令，不过**telnet只对tcp端口的监测时有用**，如果要**对udp端口的监听做测试就需要用到瑞士军刀`netcat`了**，例如：` netcat -u 127.0.0.1 9502`。其中，netcat可以简写为`nc`，即：`nc -u 127.0.0.1 9502`

12、`swoole_http_server` 继承于 `swoole_server`，是swoole内置Http服务器的支持，通过几行代码即可写出一个异步非阻塞多进程的Http服务器

> ​	注意:swoole_http_server对Http协议的支持并不完整，建议仅作为应用服务器。并且在前端增加Nginx作为代理

13、HttpServer

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/8.PNG)

通过swoole_http_response方式的话，就不能和以前的fpm模式一样使用全局数组SERVER、POST、GET，也不能使用set_cookie以及使用echo直接输出网页的响应



![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/9.PNG)



![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/10.PNG)



![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/11.PNG)

> **注意：前面那4个函数一定要在end函数的前面使用**



![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/12.PNG)

> 其他4个变量指的是：get、post、cookie、files



![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/13.PNG)



实战运用：结合先有Web框架、Nginx

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/14.PNG)

swoole擅长处理php这样的动态文件的解析，因为它不需要像fpm那样动态的创建和删除进程，而nginx更适合处理静态文件

这样配置完了之后，所有对php文件的请求就会全部代理到127.0.0.1这个IP的9501这个端口上，这个端口也就是swoole_http_server开启的监听端口

### 14、WebSocket

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/15.PNG)



实现类似与qq群群发的简单例子：

**服务器端：建立一个websocket服务器**

```php
<?php
//创建websocket服务器对象，监听0.0.0.0:9502端口
$ws = new swoole_websocket_server("0.0.0.0", 9502);

//监听WebSocket连接打开事件
$ws->on('open', function ($ws, $request) {
    $fd[] = $request->fd;
    $GLOBALS['fd'][] = $fd;
    //$ws->push($request->fd, "hello, welcome\n");
});

//监听WebSocket消息事件
$ws->on('message', function ($ws, $frame) {
    $msg =  'from'.$frame->fd.":{$frame->data}\n";
//var_dump($GLOBALS['fd']);
//exit;
    foreach($GLOBALS['fd'] as $aa){//这样就可以给所有的客户端发送消息了
        foreach($aa as $i){
            $ws->push($i,$msg);
        }
    }
   // $ws->push($frame->fd, "server: {$frame->data}");
    // $ws->push($frame->fd, "server: {$frame->data}");
});

//监听WebSocket连接关闭事件
$ws->on('close', function ($ws, $fd) {
    echo "client-{$fd} is closed\n";
});

$ws->start();
```

**客户端：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="msg"></div>
<input type="text" id="text">
<input type="submit" value="发送数据" onclick="song()">
</body>
<script>
    var msg = document.getElementById("msg");
    var wsServer = 'ws://127.0.0.1:9502';
    //调用websocket对象建立连接：
    //参数：ws/wss(加密)：//ip:port （字符串）
    var websocket = new WebSocket(wsServer);
    //onopen监听连接打开
    websocket.onopen = function (evt) {
        //websocket.readyState 属性：
        /*
        CONNECTING    0    The connection is not yet open.
        OPEN    1    The connection is open and ready to communicate.
        CLOSING    2    The connection is in the process of closing.
        CLOSED    3    The connection is closed or couldn't be opened.
        */
        msg.innerHTML = websocket.readyState;
    };

    function song(){
        var text = document.getElementById('text').value;
        document.getElementById('text').value = '';
        //向服务器发送数据
        websocket.send(text);
    }
      //监听连接关闭
//    websocket.onclose = function (evt) {
//        console.log("Disconnected");
//    };

    //onmessage 监听服务器数据推送
    websocket.onmessage = function (evt) {
        msg.innerHTML += evt.data +'<br>';
//        console.log('Retrieved data from server: ' + evt.data);
    };
//监听连接错误信息
//    websocket.onerror = function (evt, e) {
//        console.log('Error occured: ' + evt.data);
//    };

</script>
</html>
```

然后，我们把这个html文件放在nginx服务器里面的html目录里面

接着执行上面的php文件(开启websocket服务器)

这样就搭建完成了

### 15、获取某个连接来自于哪个端口

可以使用：

```php
connection_info
```

### 16、swoole_server->start

启动成功后将进入事件循环，等待客户端连接请求。`start`方法之后的代码不会执行，例如：

```php
<?php
$serv = new swoole_server('127.0.0.1', 9501, SWOOLE_BASE, SWOOLE_SOCK_TCP);

//注意：在启动服务器之前，必须得先使用receive或packet回调函数，否则会报错
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, 'Swoole: '.$data);
    $serv->close($fd);
});
var_dump('hehe');
$serv->start();
var_dump('haha');
//启动成功后将进入事件循环，等待客户端连接请求。start方法之后的代码不会执行.所以只有hehe会被打印，而haha不会被打印
```

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/17.PNG)











































































