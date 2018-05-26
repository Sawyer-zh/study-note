# Swoole入门实战打造高性能赛事直播

## 1、PHP源码安装遇到的坑

- 编译完PHP源码之后，在etc文件夹下面没有php.ini文件。

此时，需要把PHP源码包下面的`php.ini-development`拷贝到etc下面。然后修改名字为`php.ini`

- 修改完了配置文件之后，配置文件并没有生效

很大可能是因为php.ini文件的路径不是在etc下面。

可以通过命令：

```shell
php -i | grep php.ini
```

来查看：

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/1.png)

所以，我们需要把`php.ini`文件放在lib目录下面。

## 2、Swoole源码安装

我们需要先使用PHP自带的一个工具—phpize

执行完phpize之后，会生成一个configure文件。

然后再执行：

```shell
./configure --with-php-config=/usr/local/php-7.2.3/bin/php-config
```

然后在执行：

```shell
make
make install
```

然后再php.ini配置文件里面添加：

```
extension=swoole
如果无法加载动态连接库，那么可能需要加上 .so：
extension=swoole.so
```

测试扩展有没有添加成功的方式：

在swoole源码包里面的`examples/server`有一个`echo.php`文件，我们可以去执行它，然后再查看端口：

```shell
netstat -anp | grep 9501
```

如果是mac，那么使用：

```shell
lsof -i:9501
```

## 3、TCP服务器

## 4、swoole websocket服务

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/2.png)

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/3.png)

## 5、Swoole Task任务使用

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/4.png)

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/5.png)







## 6、异步非阻塞IO场景

### swoole毫秒定时器

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/6.png)

### 异步文件系统IO-读取文件



### 异步Mysql详解



### 异步Redis

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/7.png)

重新编译swoole需要使用：

```shell
make clean
make -j
make install
```

查看swoole异步的redis能否支持：

```shell
php --ri swoole
```

如果有如下：

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/8.png)

说明重新编译好的swoole支持异步redis了。

## 7、swoole创建进程



## 8、swoole内存--table

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/9.png)

swoole_table是一个**基于共享内存和锁**实现的超高性能，并发数据结构。

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/10.png)

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/11.png)

## 9、swoole 协程



## 10、swoole实战介绍

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/12.png)

### 登录流程

![](http://oklbfi1yj.bkt.clouddn.com/Swoole%E5%85%A5%E9%97%A8%E5%AE%9E%E6%88%98%E6%89%93%E9%80%A0%E9%AB%98%E6%80%A7%E8%83%BD%E8%B5%9B%E4%BA%8B%E7%9B%B4%E6%92%AD/13.png)

### 让swoole完美支持TP5

swoole对一些超全局变量（例如：`$_GET`、`$_POST`等等），不会释放。只有关闭了用swoole写的服务器之后，才会释放（因为swoole是常驻内存的）。

thinkphp它会把模块、控制器以及控制器里面的方法放在变量里面，**这些值在worker进程里面是不会被注销掉的**。所以下次访问的时候，用到的还是之前的那个模块、控制器以及控制器里面的方法。

### 基于阿里短信服务发送短信验证码

把发送短信的SDK放入thinkphp中的**extend目录下**（一般把扩展类库放到这里面）。

把验证码写入到Redis里面暂时存放起来（以便后面进行登录验证）。

注意：这个验证码是通过我们的PHP脚本的随机函数生成的，然后在把这个验证码发送给阿里大鱼，然后阿里大鱼再把验证码通过短信的方式发送给客户。

### ajax跨域问题

如果在发送ajax请求的时候，使用的url带有IP，那么需要加上协议名，例如：

```shell
http://localhost:8811/index/index/index
```

而不能是：

```shell
localhost:8811/index/index/index
```

















