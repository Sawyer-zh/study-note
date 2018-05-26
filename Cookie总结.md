# Cookie总结

## 什么是Cookie

Cookie是一种能够让网站Web服务器把少量数据储存到客户端的硬盘或内存里，或是从客户端的硬盘里读取数据的一种技术。

 Cookie文件则是指在浏览某个网站时，由Web服务器的CGI脚本创建的存储在浏览器客户端计算机上的一个小文本文件，其格式为：用户名@网站地址 ［数字］.txt。

再通俗一点的讲，由于HTTP是一种无状态的协议（不能在服务器上保持一次会话的连续状态信息。也就是**没有记录状态的能力**），服务器单从网络连接上无从知道客户身份。怎么办呢？就给客户端们颁发一个通行证，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。

![](http://oklbfi1yj.bkt.clouddn.com/Cookie%E6%80%BB%E7%BB%93/1.jpg)

## cookie的作用

保存web访问者的信息：身份识别号码ID、密码、停留的时间、用户访问该站点的次数。当用户**再次链接**Web服务器时，浏览器读取Cookie信息并传递给Web站点。

## cookie的属性

### Name

Cookie的名称

### value

该Cookie的值

### domain

可以访问该Cookie的域名

### Path

可以访问此Cookie的页面路径

### Expires/Max-Age

 该Cookie失效时间，单位秒

### Size

cookie的大小

### http

 cookie的httponly属性。若此属性为true，则只有在http请求头中会带有此cookie的信息，而不能通过js的document.cookie来访问此cookie

### secure

设置是否只能通过https来传递此条cookie

## cookie的操作

### cookie的发送

**服务器端向客户端发送**Cookie是**通过HTTP响应报文**实现的，在Set-Cookie中设置需要像客户端发送的cookie。

其中name=value是必选项

### cookie的修改与删除

Cookie并不提供修改、删除操作。如果要修改某个Cookie，只需要新建一个同名的Cookie，添加到HTTP响应报文中覆盖原来的Cookie

## Cookie的安全问题 

通常cookie信息都是**使用http连接**传递数据，这种传递方式很容易被查看，而且js里面直接有一个document.cookie方法，可以直接获取到用户的cooie,所以cookie存储的信息容易被窃取。假如cookie中所传递的内容比较重要，那么就**要求使用加密**的数据传输

### 防范cookie的安全问题

#### HttpOnly属性

设置HttpOnly属性，那么通过程序(JS脚本、Applet等)将无法读取到Cookie信息，这样能有效的防止XSS攻击

#### secure属性

当设置为true时，表示创建的 Cookie 会被以安全的形式向服务器传输，也就是只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证，如果是 HTTP 连接则不会传递该信息，所以Cookie 的具体内容不会被盗取。





















































































