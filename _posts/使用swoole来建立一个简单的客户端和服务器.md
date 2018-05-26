---
title: 使用swoole来建立一个简单的客户端和服务器(接入图灵机器人)
date: 2017-08-06 13:07:49
tags:
- swoole
- php
---

我们来用swoole这个php扩展实现一个简单的客户端和服务器，并且让它们进行简单的交流

首先，我们先来实现客户端(命名为：client.php)：

```php
<?php
$client = new swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1))
{
    exit("connect failed. Error: {$client->errCode}\n");
}

fwrite(STDOUT, 'Enter the message you want to send, and you can enter the no to quit it '."\n");
while(true) {
	fwrite(STDOUT, 'client: ');  
	$info = fgets(STDIN);
	if('no'."\n" === $info) {
		break;
	}
	$client->send($info);
	$recive = $client->recv();
	echo 'server: '.$recive."\n";
}
$client->close();
```

然后，再来实现一个简单的服务器(命名为：server.php)：

```php
<?php
$server = new swoole_server('127.0.0.1', 9501, SWOOLE_BASE, SWOOLE_SOCK_TCP);
$server->on('receive', function($server, $fd, $from_id, $data) {
	if('what is your name'."\n" === $data) {
		$server->send($fd, 'My name is huanghantao');
	}
	elseif('where are you from'."\n" === $data) {
		$server->send($fd, 'I am from China');
	}
	elseif('how old are you'."\n" === $data) {
		$server->send($fd, 'I am 20');
	}
	else{
		$apiKey = "7e97323fbb5b4c5691bc7f1ff5231b20"; 
		$apiURL = "http://www.tuling123.com/openapi/api?key=KEY&info=INFO"; 
		// 设置报文头, 构建请求报文 
		header("Content-type: text/html; charset=utf-8"); 
		$reqInfo = $data; 
		$url = str_replace("INFO", $reqInfo, str_replace("KEY", $apiKey, $apiURL)); 

		$res = json_decode(file_get_contents($url), true);
		$server->send($fd, $res['text']);
	}
});

$server->start();
```

接下来，进入这两个文件所在的目录

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8swoole%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8/1.PNG)

然后再执行这server.php文件来启动服务器：

```
php server.php
```

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8swoole%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8/2.PNG)

此时，就开始监听换回地址127.0.0.1的9501端口

接着，我们执行client.php这个文件：

```
php client.php
```

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8swoole%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8/3.PNG)

这样，客户端和服务器就建立了一个TCP网络连接

现在，我们就可以在客户端和服务器进行通信了，现在我们来试一试，很好玩的：

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8swoole%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8/4.PNG)

最后，我们输入no来让客户端主动断开与服务器的连接

![](http://oklbfi1yj.bkt.clouddn.com/%E4%BD%BF%E7%94%A8swoole%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8/5.PNG)

