# Nginx入门到实践

操作系统：centos 7

## 前期工作

### 关闭iptables规则

```shell
iptables -F
```

```shell
iptables -t nat -F
```

### 基础安装

```shell
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
```

```shell
yum -y install wget httpd-tools vim
```

## 基础篇

### Nginx的中间件架构

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/1.png)

Nginx是一个开源且高性能、可靠的HTTP中间件、代理服务。

### 为什么选择Nginx

- IO多路复用epoll

- 轻量级

  功能模块少

  代码模块化

- CPU亲和(affinity)

  也就是说不同的worker进程绑定到不同的cpu核。是一种把cpu核心和Nginx工作进程绑定的方式，把每个worker进程固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能。

- sendfile

  Nginx在处理静态文件的效率是非常有优势的。因为Nginx采用了sendfile工作的机制。

  原来的http server采用的机制如下：

  ![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/2.png)

  也就是说，文件需要经过内核空间到用户空间的拷贝，然后再从用户空间拷贝到内核空间。

  sendfile采用的机制如下：

  ![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/3.png)

  可以发现，没有内核空间到用户空间的拷贝（当然也没有用户空间到内核空间的拷贝）。这叫做零拷贝。

### 默认配置与默认站点启动

每次修改了配置之后，要记得重新启动服务器：

```shell
nginx -s reload
```

如果在重启的过程中出现了：

```Shell
nginx: [error] open() "/var/run/nginx.pid" failed (2: No such file or directory)
```

那么可以先杀死Nginx进程之后，然后重启。

一般在配置完之后，使用如下命令来检查配置是否正确：

```shell
nginx -t
```

但是上面的是检查默认的配置文件：default.conf

如果要检查指定的配置文件：

```
nginx -t -c /etc/nginx/nginx.conf
```

那么会检查`nginx.conf`目录下的所有的配置文件。

启动nginx的时候指定配置文件：

```shell
nginx -s reload -c /etc/nginx/nginx.conf
```

### Nginx变量

- HTTP请求变量 - arg_PARAMETER、http_HEADER、sent_http_HEADER

  http_HEADER的意思是http后面跟着一个http请求头的某个字段，例如：

  ```
  http_user_agent
  ```

- 内置变量 - Nginx内置的

- 自定义变量 - 自己定义


### Nginx模块讲解

#### Nginx官方模块

- `--with-http_stub_status_module`

  作用：展示Nginx的客户端状态，由于监控nginx当前的一个连接的信息。

  配置语法：

  ```
  Syntax: stub_status;
  Default: 无
  Context: server, location   // 也就是说要在server或者location下面
  ```

  例子：

  ```nginx
  location /mystatus {
      stub_status;
  }
  ```

  然后访问：主机名/mystatus，得到如下结果：

  ```
  Active connections: 1 
  server accepts handled requests
   68 68 78 
  Reading: 0 Writing: 1 Waiting: 0 
  ```

  `Active connections`表示当前活跃的连接数。

  `server accepts handled requests`中的第一个数表示Nginx处理握手总的次数；第二个数表示Nginx所处理的连接数；第三个数表示所有客户端总的请求数。

  正常情况下，第一个数和第二个数是要相等的，这样的话表示它没有丢失连接。

- `--with-http_random_index_module`

  作用：在主目录里面随机的选择一个文件（但是**不会选择隐藏文件**）作为默认的随机主页。这个模块一般很少用到。

  应用场景：比如我们想要让一个首页随机的生成，给用户展示多个不同的主页。

  配置语法：

  ```
  Syntax: random_index on | off;
  Default: random_index off; // 也就是说默认是关闭的
  Context: location  // 也就是说只能配置在location下面
  ```

- `--with-http_sub_module`

  作用：用于Nginx在给客户端响应HTTP内容的时候，用于对HTTP的内容进行替换。一般很少用。

  应用场景：比如有多个输出虚拟主机的时候，我们需要对每一个虚拟主机返回的内容进行替换或者是内容的变更。

  配置语法：

  ```
  Syntax: sub_filter string replacement;
  Default: 无
  Context: http, server, location
  ```

  或者：

  ```
  Syntax: sub_filter_last_modified on | off;
  Default: sub_filter_last_modified off;
  Context: http, server, location
  ```

   或者：

  ```
  Syntax: sub_filter_once on | off;
  Default: sub_filter_once on;
  Context: http, server, location
  ```

  例子：

  ```
  location / {
      root /wwwroot;
      index index.html index.htm;
      sub_filter '<a>imooc' '<a>IMOOC'
      sub_filter_once off;
  }
  ```

  这样会把html文件里面所有的`<a>imooc`替换为`a>IMOOC`。

- `limit_conn_module`

  作用：连接频率限制。（HTTP是建立在TCP基础上的，TCP**三次握手**叫做连接）

  ![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/4.png)

  配置语法：

  ```
  Syntax: limit_conn_zone key zone=name:size;
  Default: 无
  Context: http  // 即配置中server以外，http下
  ```

  因为需要管理连接，所以需要内存去储存连接，所以这里有zone的概念。如果我们以客户端的ip作为key，那么我们就是以客户端的ip作为限制。name是这块zone的名字，以方便被下面的`limit_conn`来调用。size就是这片空间的大小。

  ```
  Syntax: limit_conn zone number;
  Default: 无
  Context: http, server, location
  ```

  ​

- `limit_req_module`

  作用：请求频率限制。（TCP三次握手完之后**发送数据**叫做请求。当今的HTTP，允许在一个连接的基础上发送多次请求）

  配置语法：

  ```
  Syntax: limit_req_zone key zone=name:size rate=rate;
  Default: 无
  Context: http
  ```

  rate是请求的速率限制是多大。通常是 xxx个/秒。

  ```
  Syntax: limit_req zone=name [burst=number] [nodelay];
  Default: 无
  Context: http, server, location
  ```

##### Nginx的访问控制

###### 基于IP的访问控制

`http_access_module`

```
Syntax: allow address | CIDR | unix: | all;
Default: 无
Context: http, server, location, limit_except
```

```
Syntax: deny address | CIDR | unix: | all;
Default: 无
Context: http, server, location, limit_except
```

例子：

```nginx
location ~ ^/admin.html {
    root    /opt/app/code;
    deny    183.216.3.119;
    allow   all;
    index   index.html index.htm;
}
```

其中字符`~`是URL匹配，表示表示**正则匹配**。

也就是说，183.216.3.119这个ip不可以访问`admin.html`这个页面，会得到403状态码（但是这个ip可以访问其他页面），但是其他的ip可以访问这个页面。

###### 局限性

只能通过`$remote_addr`控制信任。

http_access_module是基于客户端的ip，但是对于Nginx来说它不认为哪一个是客户端，哪一个是真正的客户端。也就是说，如果我们的访问不是客户端与服务端之间连接，而是通过了一层代理（例如Nginx、7lay LSB、CDN）来实现，这样就会出现一个问题如下图片所示的问题：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/7.png)

这个时候，remote_addr就识别客户端是IP2，而不是IP1。而Nginx本来是想对IP1做限制的。

为了解决这个问题，可以使用`http_x_forwarded_for`来判断：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/8.png)

也就是说，`http_x_forwarded_for`里面记录了如下信息：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/9.png)

但是`x_forwarded_for`是协议要求的，不一定所有的CDN厂商等代理厂商会按照这个要求来做。这个是可以被修改的。

所以为了解决这个问题，还有如下方法：

- 结合geo模块
- 通过HTTP自定义变量传递

###### 基于用户的信任登录

`http_auth_basic_module`

###### 局限性

- 用户信息依赖文件方式

- 操作管理机械，效率低下

  解决方案

  - Nginx结合LUA实现高效验证
  - Nginx和LDAP打通，利用nginx-auth-ldap模块

#### 第三方模块

## 场景实践篇

### 场景Nginx中间架构

- 静态资源WEB服务
- 代理服务
- 负载均衡调度器SLB
- 动态缓存

#### 静态资源WEB服务

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/10.png)

##### 静态资源类型

非服务器动态运行生成的文件。**直接**可以在文件系统里面找到的资源。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/11.png)

##### 1、静态资源服务场景--CDN

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/12.png)

我们在请求静态资源的时候往往会用到CDN这项技术。

如上图，如果一个北京的用户想要请求一个文件，但是这个文件是放在新疆，也就是图中的资源储存中心。但是北京离新疆很远，如果直接去请求新疆的文件，那么需要的时间相对来说要比较久。

然后使用CDN分发网络的技术，把资源进行分发，发送给不同的地区，每个地区有一个代理来保存资源。

对于北京的用户来说，通过智能DNS技术把请求动态的定位到北京的这个代理上面进行请求，就不需要去请求新疆的资源了。所以省了很多时间。

##### 配置语法

文件读取：

```
Syntax: sendfile on | off;
Default: sendfile off;
Context: server, location, if in location
```

配置语法--tcp_nopush

```
Syntax: tcp_nopush on | off;
Default: tcp_nopush off;
Context: http, server, location
```

作用：**sendfile开启的情况下**（也就是说必须要在sendfile开启的情况下才能使用），提高网络包的传输效率。nopush的意思就是不是很着急需要把这个包推送给客户端。也就是把多个包一次性整体发送给客户端。

在**大文件传输的时候推荐打开**。

配置语法—tcp_nodelay

```
Syntax: tcp_nodelay on | off;
Default: tcp_nodelay on;
Context: http, server, location
```

作用：**在keepalive连接下**（即长连接），提高网络包的传**输实时性**。

配置语法--压缩

```
Syntax: gzip on | off;
Default: gzip off;
Context: http, server, location, if in location
```

作用：压缩传输。使用压缩可以减少网络传输的消耗。可以减少服务端的带宽资源。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/13.png)

压缩是在服务端，解压是在浏览器（现在绝大多数浏览器支持通用协议的解压功能）。

压缩比：

```
Syntax: gzip_comp_level level;
Default: gzip_comp_level 1;
Context: http, server, location
```

控制gzip http协议的版本：

```
Syntax: gzip_http_version 1.0 | 1.1
Default: gzip_http_version 1.1
Context: http, server, location
```

例子：

有一个`css`文件：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/14.png)

大小大概是23KB。

配置如下：

```
location ~ .*\.(txt|xml|css)$ {
    gzip    off;
    gzip_http_version    1.1;
    gzip_comp_level    6;
    gzip_types    text/plain application/javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
    root /opt/app/code/doc;
}
```

此时`gzip    off`是关闭的状态：

得到结果：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/15.png)

在`Size`那一个字段下面，分别显示：24.0KB和23.8KB。重点关注24.0KB。这个和压缩有关。

现在开启压缩：

```
location ~ .*\.(txt|xml|css)$ {
    gzip    on;
    gzip_http_version    1.1;
    gzip_comp_level    6;
    gzip_types    text/plain application/javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
    root /opt/app/code/doc;
}
```

得到结果：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/16.png)

可以发现24.0KB变成了3.4KB。说明压缩效果还是很明显的。

现在让压缩比例变小：

```
gzip_comp_level    1;
```

得到结果：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/17.png)

3.4KB增加到了4.2KB，说明压缩效果变差了点。

##### 2、浏览器缓存

HTTP协议定义的缓存机制（如：Expires；Cache-control等）

实现浏览器的缓存依靠特殊的头信息来和服务端进行特殊的验证。

有了浏览器的缓存，客户端就不会每次请求都请求服务端，以免给服务端造成一些资源的消耗。因为有了浏览器的缓存，客户端从本地就可以读取到资源。

###### 浏览器无缓存的情况

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/18.png)

###### 客户端有缓存的情况

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/19.png)

浏览器这个时候会先去看一下自己的缓存里面有没有这个文件，如果有，那么还要看看是否过期，如果没有过期，那么浏览器直接从缓存中获取文件即可，如果已经过期，那么客户端需要向服务器进行一系列的校验机制，校验完成以后，判断是否有新的文件，如果有新的文件更新，那么服务器就会发送新的文件给客户端，如果没有的话，客户端依然可以从缓存中读取。

###### 验证过期机制

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/20.png)

这里验证是否过期验证的是本地缓存是否过期。通过在本地缓存的文件中的HTTP头信息的`Expires`和`Cache-Control`来验证。它们的区别是协议版本，`Expires`是HTTP1.0，`Cache-Control`是HTTP1.1版本。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/21.png)

###### 配置语法

```
Syntax: expires [modified] time;
         expires epoch | max | off;
Default: expires off;
Context: http, server, location, if in location
```

配置完后，会在服务器的响应头里面添加`Cache-Control`、`Expires`头

例子：

```nginx
location ~ .*\.(htm|html)$ {
    #expires    24h;
    root    /opt/app/code;
}
```

浏览器访问，得到结果：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/22.png)

status字段是200。

刷新页面，得到结果：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/23.png)

status字段变成了304。

出现304的原因是服务端与客户端的请求进行了一个缓存的`Last-Modified`和`ETag`校验。如果发现缓存没有更新，那么服务端会返回304。

但是按照上面的流程图，应该是直接从本地缓存读取文件即可，但是为什么会得到服务器返回的响应呢？

因为在`Request Header`里面，有一个`Cache-Control: max-age=0 `：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/24.png)

但是我们的服务器并没有返回max-age给客户端。在请求头里面出现这个信息是浏览器自己加进去的（谷歌浏览器这么做，其他浏览器不一定）。目的是每一次请求都要去向客户端进行一次校验，看服务端给的`Last-Modified`是否和服务端时间更新时间一样，如果不一样，说明文件有更新，返回状态码200。如果一样，说明文件没有更新，则返回304。

例如此时我们对文件进行修改，然后刷新：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/25.png)

可以看到，文件的更新，使得状态码变成了200。

此时开启`expires`：

```nginx
location ~ .*\.(htm|html)$ {
    expires    24h;
    root    /opt/app/code;
}
```

清空下浏览器缓存，此时去第一次请求文件：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/26.png)

查看响应头：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/27.png)

发现多了`Cache-Control`字段，而且值刚好是24小时（86400秒等于24小时）。也就是说服务器告诉浏览器要按照这个缓存规则。

此时继续刷新页面，结果如下：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/28.png)

返回状态码304，说明客户端又去向服务器请求了，也就是说并没有从本地的缓存中获取文件。也就是说`Cache-Control`并没有起作用。我们看看客户端的请求头：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/29.png)

发现`Cache-Control`的`max-age=0`，说明**客户端**并没有去理会服务器返回的这个值，也就是说没有遵循服务器定的规则，而是让它直接为0。所以每次客户端都需要去请求服务器。

浏览器这么做的目的是可以实时的去和服务器进行交互，确认文件是否有更新，如果有更新的话，就能实时的发现。

##### 3、跨域访问

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/30.png)

也就是说有一个页面里面，请求`www.a.com`的时候又去请求`www.b.com`。这样就会出现一个页面请求两个不同的不同的站。出于安全考虑，浏览器禁止这么做。

###### 禁止原因

不安全，容易出现`CSRF`攻击，即**跨站式攻击**。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/31.png)

跨站式攻击原理：

这个用户去正常访问网站A，网站A返回一些信息（例如cookie）给用户，如果用户一不小心点到了网站B（非法的网站），这个时候，黑客就可以给用户发送一些恶意的东西，然后再让用户去请求网站A，这样就形成了跨站式攻击。

###### 配置语法

为了让浏览器允许跨站访问，需要服务器给客户端（浏览器）返回一些响应头，即`Access-Control-Allow-Origin`

```
Syntax: add_header name value [always]
Default: 无
Context: http, server, location, if in location
```

##### 4、防盗链

目的：防止资源被盗用。

因为在正常情况下，我们希望一些合法的用户来获取资源，并不希望所有用户（例如竞争对手）能够获取资源。

###### 防盗链设置思路

首要方式：区别哪些请求是非正常的用户请求

###### 基于http_refer防盗链配置模块

refer是请求某个资源的上一个url，例如，有一个refer.html页面：

```
<html>
<img src="http://106.14.2.109/1.png"></img>
</html>
```

我们去请求这个页面：`http://106.14.2.109/refer.html`，那么此时refer.html的refer是没有的，因为我们是直接打开这个页面的；而`http://106.14.2.109/1.png`的refer是`http://106.14.2.109/refer.html`。因此，可以通过refer来判断是否是我们这个网站的。

```
Syntax: valid_referers none | blocked | server_names | string ...;
Default: 无
Context: server, location
```

例子：

```nginx
valid_referers none blocked 106.14.2.109;
if ($invalid_referer) {
    return 403;
}
```

（`valid_referers`也支持正则匹配）

其中，`none`表示允许没有带refer信息的请求，`blocked`表示允许refer信息不是标准的 http://域名 这种方式的请求，`106.14.2.109`表示允许refer信息是这个ip的请求。

可以通过`curl`命令来进行测试。curl命令可以带有refer参数。

#### 代理服务

##### 代理服务

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/32.png)

正向代理：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/33.png)

例如翻墙软件。

反向代理：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/34.png)

代理区别：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/35.png)

##### 配置语法

```
Syntax: proxy_pass URL;
Default: 无
Context: location, if in location, limit_except
```

其中：`proxy_pass URL`表示请求到达nginx这个代理服务器之后，然后nginx再去请求这个URL。

常用的URL配置方式：

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/36.png)

反向代理例子：

其中一台nginx服务器的配置如下：

```nginx
location / {
    root    /usr/share/nginx/html;
    index    index.html index.htm;
}

location ~ /test_proxy.html$ {
    proxy_pass http://127.0.0.1:8080;
}
```

这个nginx服务器**正在监听80端口**。并且把请求反向代理到`http://127.0.0.1:8080`。

假设有一个的nginx服务器正在监听8080端口，并且这个8080端口是公网无法访问到的。此时我们需要去请求正在监听80端口的Nginx服务器，然后把请求转发给正在监听8080端口的Nginx服务器。

监听8080端口的那个Nginx服务器配置如下：

```nginx
location / {
    root    /opt/app/code2;
    index    index.html index.htm;
}
```

在目录`/opt/app/code2`下有一个`proxy.html`文件。

此时我们去请求：

```http
http://ip/proxy.html
```

会得到页面内容。也就是说变成了：

```http
http://ip:8080/proxy.html
```

因为是服务器本地去访问8080端口，所以可以请求成功。

如果我们把反向代理关了再去请求：

```http
http://ip/proxy.html
```

那么就会得到404，因为这个页面不在`/usr/share/nginx/html`下面。

#### 负载均衡服务

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/37.png)

需要负载均衡的原因：

最开始的部署模型往往是最简单的部署模型，即点对点的服务。当随着企业业务的增长带来的客户海量的请求，给我们的服务端就造成了很大的并发访问，此时服务器响应可能不能及时。此时我们就需要不断的扩容我们的后端服务。那么对于我们前端就需要有一个负载均衡来均分这个请求，提升整体后端的吞吐率。所以对于请求而言，负载均衡就可以很好的均摊请求。

另一方面，对于点对点的方式，如果一个点宕掉了，整体的服务就会挂掉。但是有了负载均衡，即使某个点挂了，其他的点还能用。只要把挂了的点剔除即可，这样更加高可用。

##### 负载均衡分类

按照范围分类。

###### GSLB

全局负载均衡

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/38.png)

它的节点非常庞大，地域范围比较广，往往是以国家或省为单位进行全局的负载均衡。

###### SLB

往往调度节点和服务节点在一个逻辑单元里面或在一个地域里面。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/39.png)

除了按照地域划分以外，按照网络的模型（即OSI）可以分为四层负载均衡和七层负载均衡。

###### 四层负载均衡

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/40.png)

###### 七层负载均衡

Nginx就是典型的7层负载均衡。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/41.png)

##### Nginx负载均衡

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/42.png)

nginx实现负载均衡用到了`proxy_pass`，但是不是转发到一台服务器上而是一组服务池，称为upstream server。服务池可以定义所有的服务器单元，这些服务器都可以提供相同的服务，所以放到一个服务池里面。然后对服务池里面的服务器进行请求的轮询，实现负载均衡。

###### 配置语法

```
Syntax: upstream name { ... }
Default: 无
Context: http  // 即必须在http层以下，server层以外
```

例子：

负载均衡服务器配置：

```nginx
upstream hht {
    server    116.62.103.228:8001;
    server    116.62.103.228:8002;
    server    116.62.103.228:8003;   
}
```

```nginx
server {
    listen    80;
    ...
    
    location / {
        proxy_pass    http://hht; // 也就是把所有请求都代理到http://hht
        include    proxy_params;
    }
}
```

当访问这个负载均衡服务器的时候，会去循环轮询请求服务器池的那3个nginx服务器。

###### upstream举例

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/43.png)

weight代表权重，也就是访问这台服务器的概率。

###### 后端服务器在负载均衡调度器中的状态

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/44.png)

其中backup也是不提供服务。

###### 调度算法

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/45.png)

#### 缓存服务

目的：减少服务端的压力。

##### 缓存类型

###### 服务端缓存

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/46.png)

服务端缓存最常见的是Redis、memcache等等储存key-value这样的数据。

###### 代理缓存

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/47.png)

当缓存放在了代理上面的时候，称为代理缓存。它的内容是从服务端获取到的，然后再缓存到代理上面。

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/49.png)

###### 客户端缓存

![](http://oklbfi1yj.bkt.clouddn.com/Nginx%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E8%B7%B5/48.png)




