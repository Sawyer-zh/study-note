# AJAX原理总结

### AJAX全称

> Asynchronous JavaScript and XML(异步的JavaScript 和XML)

### 同步和异步 异步传输是面向字符的传输，单位是字符

同步传输是面向比特，单位是帧，传输时要求接收方和发送方的时钟是保持一致的。

### 通过XMLHTTPRequest理解AJAX

> AJAX原理简单地说就是通过XMLHTTPRequest来向服务器发送异步请求，从服务器获得数据，然后用JavaScript来操作DOM而刷新页面。

#### XMLHTTPRequest对象属性 

![](http://oklbfi1yj.bkt.clouddn.com/AJAX%E5%8E%9F%E7%90%86%E6%80%BB%E7%BB%93/1.png)

AJAX就是把JavaScript技术与XMLHTTPRequest对象放在WEB表单和服务器间，当用户向服务器请求时，数据发送给JavaScript代码而不是直接发给服务器，JavaScript代码在幕后发送异步请求，然后服务器将数据返回给JavaScript代码。JavaScript代码接收到数据后，操作DOM来更新页面数据。

### AJAX同步和异步

异步：AJAX一直执行，不发生阻塞等待服务器的响应，因此在使用异步AJAX时，在函数中对变量赋值是没有效的。

同步：AJAX发生阻塞，等待服务器返回数据才能执行下一步。

![](http://oklbfi1yj.bkt.clouddn.com/AJAX%E5%8E%9F%E7%90%86%E6%80%BB%E7%BB%93/2.png)

（记住这幅图即可）

### AJAX跨域请求----JSONP

#### JSONP是什么

是资料格式 JSON 的一种“使用模式”，可以让网页从别的网域要资料。可以理解成填充了内容的json格式数据。

#### 跨域是什么

简单理解，当a.com/get.html文件需要获取b.com/data.html文件中的数据，而这里a.com和b.com并不是同一台服务器，这就是跨域。

#### JSONP是怎么产生的

因为AJAX无法实现跨域请求，凡事WEB页面中调用JavaScript文件不受是否跨域影响。于是判断，是否能通过纯WEB端跨域访问数据，只有一种可能，即在远程服务器上 设法把数据装入JavaScript格式的文件里，供客户端调用和进一步处理。因为JSON格式可以很方便地在客户端和服务器使用，所以有以下方案： WEB客户端通过与调用脚本一模一样的方式，来调用跨域服务器上动态生成的JavaScript格式文件，显而易见，服务器之所以要动态生成JSON文件，目的就 在于把客户端需要的数据装入进入。客户端接收到数据后就按照自己的需求进行处理和展现。久而久之，为了便于客户端使用数据，逐渐地形成一个非正式 传输协议，成为JSONP。

#### 基本原理

简单地说就是动态添加一个<script>标签，而**script标签的src属性是没有跨域的限制的**。

#### 要点

允许用户传递一个callback参数给服务端，服务端返回数据时会将这个callback参数作为函数名来包裹JSON数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。

#### 缺点

只支持GET请求。

#### 执行过程

在客户端注册一个callback函数，然后把callback的名字传给服务器。客户端接收到服务器返回的callback函数后，以JavaScript语法的形式生成给一个function，function名字就是服务器返回的参数的值。 最后将JSON数据直接以入参的方式，放置到function中，这样就生成了一堆JavaScript代码，返回给客户端。

