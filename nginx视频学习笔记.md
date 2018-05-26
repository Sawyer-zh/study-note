# nginx视频学习笔记

### 1、环境调试确认

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1.PNG)

确认是否连接网络的方法可以使用：

```
ping www.baidu.com
```

可以通过下面的命令来查看是否有`iptables`规则：

```
iptables -L
```

如果有，可以通过下面命令来关闭`iptables`规则：

```
iptables -F
```

可以通过下面的命令来查看`selinux`是否开启：

```
getenforce
```

如果是`Disabled`说明没有开启，如果开启了，可以使用下面的命令关闭：

```
 setenforce 0
```

### 2、两项安装

```
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
```

```
yum -y install wget httpd-tools vim
```

### 3、一次初始化

```
cd /opt;mkdir app download logs work backup
```

`app`是代码目录

`download`下面放下载的源码包

`logs`下面放自定义的日志

`work`下面放自定义的脚本

`backup`下面放默认的配置文件的备份

### 4、中间件

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/2.PNG)

中间件除了上面这张图片里面的作为中介的作用之外，还可以实现：

对于web请求的负载均衡

对于http请求的缓存服务

还可以实现安全应用防控的设置

### 5、Nginx简述

Nginx是一个开源且性能高、可靠的HTTP中间件、代理服务

是HTTP的中间件的服务，也可以作为HTTP的代理的服务

### 6、为什么选择nginx

#### 优点一：IO多路复用epoll

**什么是IO复用？**	![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/3.PNG)

#### IO复用其实解决的是一个并发性的问题。比如大家所看的慕课网一样，大家都在请求慕课网，这个时候对服务端后天而言，就会产生多个请求，处理多个并发的请求对于中间件就要产生多个IO流对于系统的读写。对于IO流，请求操作系统内核有并行处理和串行处理。对于串行处理，前一个发生阻塞，后面的就没法完成请求，所以这个时候就要考虑采用并行的方式来实现最大的并发和吞吐，这个时候就用到了IO复用的技术

#### IO复用就是让一个Socket来作为复用完成整个IO流的请求

#### 其中，可以使用上图多线程的方式来实现整个IO流的请求，除了多线程的方式，还可以使用IO`多路`复用的方式来实现IO流的请求

#### 什么是IO多路复用？

​	多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用，这里的**“复用”指的是复用同一个线程**

举个形象的例子：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/4.PNG)

​	类似于一个老师布置了一道题目，然后老师使用分身术，在每个学生的旁边进行解答，这样效率更高(这种方式就是一种多线程的方式，但是多线程的处理会产生一定的消耗，因为老师分身之后，我们还要管理这多个老师)

​	而不需要一个接着一个的对学生解答(一个一个的解答类似于串行的，一个学术不会，则老师要讲解半天，耽误了其他的学生)

**所以这个时候，我们使用IO多路复用：**

也就是说，由学生主动上报。老师出完题之后，比如说学术B已经答完了，这个时候老师再过去解答问题，其他的学生在等待，因为学生B的学习效率高，他答非常快，其他的学生可能还在做题

然后去解答其他学生的问题

### 什么是epoll

IO多路复用的实现方式：select、poll、epoll

**select模型**

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/5.PNG)

select缺点：

1、select模型采用的是线性遍历的方式，这种方式存在一个问题，**它会不断的去遍历队列里面的内容**，从而效率低下

2、能够监视文件描述符(fd)的数量存在最大限制

**epoll模型**(nginx就是采用的epoll模型)

​	每当FD就绪，采用系统的回调函数之间将fd放入，效率更高

​	最大连接无限制

针对`select`和`epoll`举个例子：

select模型的例子：

我们在餐馆里面吃完饭结账的时候，我们会通知服务员我们要结账了，服务员会告诉老板(只告诉了有几桌没有结账，并没有说是哪几桌没结账)，然后老板会过来结账，但是因为不知道是哪几桌，这是后老板就要去问哪几桌

epoll模型的方式：

服务员会告诉老板是哪几号桌要结账，老板就直接到对应的几号桌去结账，这样的效率是比较高的

#### 优点二：轻量级

​	1、功能模块少(nginx只保留了跟HTTP以及相关核心功能的那一部分核心代码，另一些类似于插件式的不会集成在nginx里面)

​	2、代码模块化(易读、易于进行二次改进，对开发人员友好)

#### 优点三：CPU亲和(affinity)

CPU亲和：是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能

#### 优点四：sendfile

nignx在处理静态文件的效率是非常有优势的，这是因为nginx采用了sendfile的机制

原来的http server采用的机制的示意图：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/6.PNG)

解释：当我们去请求一个文件的时候，它要经过操作系统的内核空间和用户空间，最终到达Socket，通过Socket再将响应传递给用户

对于一台服务器而言，它要经过内核空间到用户空间，也就是要发生多次的切换。但是，其实**静态文件是不需要经过过多的一步步的空间的逻辑性的处理，直接就可以通过内核空间进行传输，sendfile正是利用到了这种模式**

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/7.PNG)

也就是Linux2.2出来的零拷贝这种传输模式

### 7、nginx安装目录讲解

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/8.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/9.PNG)

(**`nginx.conf`是nginx的主配置文件，nginx启动的时候主要会读`nginx.conf`，然后在没有变更的情况下，会读`default.conf`**)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/10.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/11.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/12.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/13.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/14.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/15.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/16.PNG)



nginx除了做http代理服务器之外，还可以做为一个缓存服务器

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/17.PNG)

对于纠错、分析非常有用的

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/18.PNG)

### 8、nginx安装编译参数

命令：

```
nginx -V
```

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/19.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/20.PNG)



虽然我们是用root用户启动的nginx，但是nginx出于安全性的考虑，nginx真正的工作进程(也就是它的worker)是用nginx这个系统用户来跑的：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/21.PNG)

### 9、nginx默认配置语法

我们打开主配置文件`/etc/nginx/nginx.conf`，然后找到文件里面的那个`http`下面的那个`include /etc/nginx/conf.d/*.conf`：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/22.PNG)

也就是说这个主配置文件`nginx.conf`它包含了那个目录下的所有`*.conf`配置文件(也就是说nginx会先去读主配置文件，当读到这个include的时候，nginx也会把被包含的那些配置文件也读下来。我们可以把被包含进来的文件叫做**子配置文件**，里面会放一些子server)

`nginx.conf`文件里面有一个`http`模块：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/23.PNG)

按照层级来划分，我们可以看到`http`是最外一层，所有的http我都可以包含多个`http服务`

一个server配置一个独立的、虚拟的站点，在每个server里面，配置当前server的属性

其中的`listen`表示监听那个端口

`server_name`默认写的是`localhost`，如果有虚拟主机，独立域名的，则可以把自己的主机名或者对外的域名填入到server_name那个地方	

`location`是`server_name`下面的一个模块，他主要是控制着每一层的路径的访问

`error_page`用来**定义**当服务端返回表示错误的状态码的时候，统一定义一个错误页面，然后这个页面需要匹配一个`location`也就是图中的`location = /50x.html`，而这个页面所在的路径是`root /usr/share/nginx/html`(这里的root指的是这个页面所在的路径)

10、默认配置与默认站点启动

因为在主配置文件里面，我们引入了`/etc/nginx/conf.d/*.conf`这些所有`.conf`配置文件，所以，我们可以在`/etc/nginx/conf.d/`目录下面新建一个文件，取名为：`default.conf`，此时，这个子配置文件会被主配置文件引入，我们在`default.conf写入`：

```nginx
server {
    listen      80;
    server_name localhost;

    location / {
        root    /usr/share/nginx/html;
        index   index.htmld  index.htm
    }

    error_page  500 502 503 504 /50x.html;
    location = /50x.html {
        root    /usr/share/nginx/html;
    }

}
```

然后我们在浏览器里面输入主机名或者localhost即可访问到index.html文件

查看主机的方法是在终端里面输入：

```
ip a
```

会有：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/24.PNG)

划红线部分就是主机ip

#### 注意：当修改了nginx的配置之后，要重启nginx才能有效，重启的命令：

```
service nginx restart
```

这里的`service`是一个脚本程序(/usr/sbin/service)，它为 /etc/init.d/目录下的众多服务器程序(httpd、vsftpd、sshd和mysqld等)的启动(start)、停止(stop)和重启(restart)等动作提供了一个统一的管理

































































































