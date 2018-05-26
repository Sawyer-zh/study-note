# 图解Http学习笔记

1、根据 Web 浏览器地址栏中指定的 URL，Web 浏览器从 Web 服务器端获取文件资源（resource）等信息，从而显示出 Web 页面 

​	**像这种通过发送请求获取服务器资源的 Web 浏览器等，都可称为客户端（client）** 

2、HTTP 通常被译为超文本传输协议，但这种译法并不严谨。严谨的译名应该为“超文本转移协议” 

3、构建WWW的3 项技术  ：HTML 、HTTP 、URL 

4、当年 HTTP 协议的出现主要是为了解决文本传输的难题 。现在 HTTP 协议已经超出了 Web 这个框架的局限，
被运用到了各种场景里 

5、不同的硬件、操作系统之间的通信，所有的这一切都需要一种规则。而我们就把这种规则称为协（protocol） 

6、网络层（又名网络互连层）
网络层用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位。该层规定了通过怎样的路径（所谓的传输路线）到达对方计算机，并把数据包传送给对方 

与对方计算机之间通过多台计算机或网络设备进行传输时，网络层所起的作用就是在众多的选项内选择一条传输路线 

7、链路层（又名数据链路层，网络接口层）

用来处理连接网络的硬件部分。包括控制操作系统、硬件的设备驱动 。硬件上的范畴均在链路层的作用范围之内 

8、

我们用 HTTP 举例来说明，首先作为发送端的客户端在应用层（HTTP 协议）发出一个想看某个 Web 页面的 HTTP 请求。

接着，为了传输方便，在传输层（TCP 协议）把从应用层处收到的数据（HTTP 请求报文）进行分割，并在各个报文上打上标记序号及端口号后转发给网络层。
在网络层（IP 协议），增加作为通信目的地的 MAC 地址后转发给链路层。这样一来，发往网络的通信请求就准备齐全了。
接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用层。当传输到应用层，才能算真正接收到由客户端发送过来的 HTTP 请求。 

### (发送端是每通过一层则增加首部，接收端每通过一层则删除首部)

9、与 HTTP 密不可分的 3 个协议：IP、TCP 和 DNS

10、IP和IP地址是两码事：IP 协议的作用是把各种数据包传送给对方。而**要保证确实传送到对方那里，则需要满足各类条件。其中两个重要的条件是 IP 地址和 MAC 地址** 

IP 地址指明了节点被分配到的地址，MAC 地址是指网卡所属的固定地址。IP 地址可以和 MAC 地址进行配对。IP 地址可变换，但 MAC 地址基本上不会更改 

**IP 间的通信依赖 MAC 地址** 

11、为了准确无误地将数据送达目标处，TCP 协议采用了三次握手策略。用 TCP 协议把数据包送出去后，TCP 不会对传送后的情况置之不理，它一定会向对方确认是否成功送达。握手过程中使了 TCP 的标志（flag） —— SYN（synchronize） 和 ACK（acknowledgement）。发送端首先发送一个带 SYN 标志的数据包给对方。接收端收到后，回传一个带有 SYN/ACK 标志的数据包以示传达确认信息。**最后，发送端再回传一个带 ACK 标志的数据包，代表“握手”结束**。若在握手过程中某个阶段莫名中断，TCP 协议会再次以相同的顺序发送相同的数据包 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1.PNG)

12、用户通常使用主机名或域名来访问对方的计算机，而不是直接通过 IP 地址访问 。**但要让计算机去理解名称相对而言就变得困难了。因为计算机更擅长处理一长串数字 。所以，DNS 服务应运而生** 

​	DNS 协议提供通过域名查找 IP 地址，或逆向从 IP 地址反查域名的服务 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/2.PNG)

13、URI 用字符串标识某一互联网资源，而 URL 表示资源的地点（互联网上所处的位置）。可见 URL 是 URI 的子集 

14、列举了几种 URI例子 ：

`ftp://ftp.is.co.za/rfc/rfc1808.txt`

`http://www.ietf.org/rfc/rfc2396.txt`

`ldap://[2001:db8::7]/c=GB?objectClass?one `

`telnet://192.0.2.16:80/`

15/即使是相同的 HTML 文档，通过改变应用的 CSS，用浏览器看到的页面外观也会随之改变。CSS 的理念就是让文档的结构和设计分离，达到解耦的目的 

16、CGI（Common Gateway Interface，通用网关接口）是**指 Web服务器在接收到客户端发送过来的请求后转发给程序的一组机制**。在 CGI 的作用下，程序会对请求内容做出相应的动作，比如创建 HTML 等动态内容。使用 CGI 的程序叫做 CGI 程序，通常是用 Perl、PHP、Ruby 和C 等编程语言编写而成 

​	CGI，由于每次接到请求，程序都要跟着启动一次。因此一旦访问量过大，Web 服务器要承担相当大的负载 

17、从整体上看，HTTP 就是一个通用的单纯协议机制。因此它具备较多优势，但是在安全性方面则呈劣势 

​	就拿远程登录时会用到的 SSH 协议来说，SSH 具备协议级别的认证及会话管理等功能，HTTP 协议则没有 

​	**因此，开发者需要自行设计并开发认证及会话管理功能来满足 Web 应用的安全** 

18、在两台计算机之间使用 HTTP 协议通信时，在一条通信线路上必定有一端是客户端，另一端则是服务器端 

19、

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/3.PNG)

20、HTTP 是一种不保存状态，即无状态（stateless）协议 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/4.PNG)

这是为了更快地处理大量事务，确保协议的可伸缩性，而特意把 HTTP 协议设计成如此简单的 

HTTP/1.1 虽然是无状态协议，但为了实现期望的保持状态功能，于是引入了 Cookie 技术。有了 Cookie 再用 HTTP 协议通信，就可以管理状态了 

21、HTTP/1.1 中可使用的方法 

​	**GET ：获取资源** 

​	GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。也就是说，如果请求的资源是文本，那就保持原样返回；如果是像 CGI（Common Gateway Interface，通用网关接口）那样的程序，则返回经过执行后的输出结果 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/5.PNG)

​	**POST：传输实体主体** 

​	虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行传输，而是用 POST 方法。虽说 POST 的功能与 GET 很相似，但 POST 的主要目的并不是获取响应的主体内容 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/6.PNG)

​	**PUT：传输文件**
​	PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置 

​	**HEAD：获得报文首部**
​	HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/7.PNG)

​	**DELETE：删除文件**
​	DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE方法按请求 URI 删除指定的资源 

​	**OPTIONS：询问支持的方法**
​	OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/8.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/9.PNG)

22、HTTP 协议的初始版本中，每进行一次 HTTP 通信就要断开一次TCP 连接 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/10.PNG)

以当年的通信情况来说，因为都是些容量很小的文本传输，所以即使这样也没有多大问题。可随着 HTTP 的普及，文档中包含大量图片的情况多了起来 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/11.PNG)

为解决上述 TCP 连接的问题，HTTP/1.1 和一部分的 HTTP/1.0想出了持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或 HTTP connection reuse）的方法。持久连接的特点是，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/12.PNG)

### 23、管线化

持久连接使得多数请求以管线化（pipelining）方式发送成为可能 

从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求 。这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待响应了 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/13.PNG)

### 24、Cookie

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie值后发送出去 

​	**没有 Cookie 信息状态下的请求** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/14.PNG)

**第 2 次以后（存有 Cookie 信息状态）的请求** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/15.PNG)

1. 请求报文（没有 Cookie 信息的状态）

   ```http
   GET /reader/ HTTP/1.1
   Host: hackr.jp
   *首部字段内没有Cookie的相关信息
   ```

2. 响应报文（**服务器端生成** Cookie 信息）

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 12 Jul 2012 07:12:20 GMT
   Server: Apache
   ＜Set-Cookie: sid=1342077140226724; path=/; ex
   pires=Wed,
   10-Oct-12 07:12:20 GMT＞
   Content-Type: text/plain; charset=UTF-8
   ```

3. 请求报文（**自动发送保存着的 Cookie 信息**）

   ```http
   GET /image/ HTTP/1.1
   Host: hackr.jp
   Cookie: sid=1342077140226724 
   ```

   ### 25、HTTP 报文

   用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文，响应端（服务器端）的叫做响应报文 

   HTTP 报文大致可分为报文首部和报文主体两块 

   ​	**报文首部**是指：服务器端或客户端需处理的请求或响应的内容以及属性

   ​	**报文主体**是指：应被发送的数据

   ​

   ​	**报文（message）**
   ​		是 HTTP 通信中的基本单位，由 8 位组字节流（octet sequence，其中 octet 为 8 个比特）组成，通过 HTTP 通信传输。
   ​	**实体（entity）**
   ​		作为请求或响应的有效载荷数据（补充项）被传输，其内容由
   实体首部和实体主体组成 


### 26、分块传输编码

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/16.PNG)

### 27、发送多种数据的多部分对象集合 

发送邮件时，我们可以在邮件里写入文字并添加多份附件。这是因为采用了 MIME（Multipurpose Internet Mail Extensions，多用途因特网邮件扩展）机制，它允许邮件处理文本、图片、视频等多个不同类型的数据。 

### 多部分对象集合包含的对象如下。

`multipart/form-data`：在 Web 表单文件上传时使用。

`multipart/byteranges`：状态码 206（Partial Content，部分内容）响应报文包含了多个范围的内容时使用 

**在 HTTP 报文中使用多部分对象集合时，需要在首部字段里加上Content-type** 

28、获取部分内容的范围请求 

以前，用户不能使用现在这种高速的带宽访问互联网，当时，下载一个尺寸稍大的图片或文件就已经很吃力了。如果下载过程中遇到网络中断的情况，那就必须重头开始。为了解决上述问题，需要一种可恢复的机制。所谓恢复是指能从之前下载中断处恢复下载。要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请求叫做范围请求（Range Request）。对一份 10 000 字节大小的资源，如果使用范围请求，可以只请求5001~10 000 字节内的资源 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/17.PNG)

针对范围请求，响应会返回状态码为 **206 Partial Content** 的响应报文 

如果服务器端无法响应范围请求，则会返回状态码 200 OK 和完整的实体内容 

### 29、内容协商返回最合适的内容 

同一个 Web 网站有可能存在着多份相同内容的页面。比如英语版和中文版的 Web 页面，它们内容上虽相同，但使用的语言却不同。 当浏览器的默认语言为英语或中文，访问相同 URI 的 Web 页面时，则会显示对应的英语版或中文版的 Web 页面。这样的机制称为**内容协商** 

内容协商机制是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。内容协商会以响应资源的语言、字符集、编码方式等作为判断的基准。包含在请求报文中的某些首部字段（如下）就是判断的基准 

### 30、状态码

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/18.PNG)

状态码如 200 OK，以 3 位数字和原因短语组成 

数字中的第一位指定了响应类别，后两位无分类。响应类别有以下 5 种：

 ![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/19.PNG)

**具有代表性的几个状态码 :**



**200 OK**

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/20.PNG)



**204 No Content**

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/21.PNG)

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如，当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面不发生更新 



**206 Partical Content**

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/22.PNG)



#### 3XX 响应结果表明浏览器需要执行某些特殊的处理以正确处理请求 



**301 Moved Permanently** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/23.PNG)

永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI 保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存 



**304 Not Modified** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/24.PNG)

该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应的主体部分。**304 虽然被划分在 3XX 类别中，但是和重定向没有关系** 

#### 4XX 的响应结果表明客户端是发生错误的原因所在 

**400 Bad Request** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/25.PNG)

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。另外，浏览器会像 200 OK 一样对待该状态码 



**401 Unauthorized** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/26.PNG)



**403 Forbidden** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/27.PNG)

该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因 



**404 Not Found** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/28.PNG)

该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用 

#### 5XX 的响应结果表明服务器本身发生错误 

**500 Internal Server Error** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/29.PNG)

该状态码表明服务器端在执行请求时发生了错误。也有可能是 **Web 应用存在的 bug** 或某些临时的故障 



**503 Service Unavailable** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/30.PNG)

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入 RetryAfter 首部字段再返回给客户端 

### 31、状态码和状况的不一致

不少返回的状态码响应都是错误的，但是用户可能察觉不到这点。比如 Web 应用程序内部发生错误，状态码依然返回 200 OK，这种情况也经常遇到 

### 32、状态码的工作机制 

### 33、用单台虚拟主机实现多个域名 

HTTP/1.1 规范允许一台 HTTP 服务器搭建多个 Web 站点。比如，提供 Web 托管服务（Web Hosting Service）的供应商，可以用一台服务器为多位客户服务，也可以以每位客户持有的域名运行各自不同的网站。这是因为利用了虚拟主机（Virtual Host，又称虚拟服务器）的功能 

域名通过 DNS 服务映射到 IP 地址（域名解析）之后访问目标网站。可见，当请求发送到服务器时，已经是以 IP 地址形式访问了 

### 34、代理、网关、隧道

**代理**
代理是一种有转发功能的应用程序，它扮演了位于服务器和客户端“中间人”的角色，接收由客户端发送的请求并转发给服务器，同时也接收服务器返回的响应并转发给客户端。

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/31.PNG)

使用代理服务器的理由有：利用缓存技术（稍后讲解）减少网络带宽的流量 

**网关**
网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请求时，它就像自己拥有资源的源服务器(持有资源实体的服务器被称为源服务器 )一样对请求进行处理。有时客户端可能都不会察觉，自己的通信目标是一个网关。网关的工作机制和代理十分相似。而网关能使通信线路上的服务器提供非 HTTP 协议服务 

利用网关能提高通信的安全性，因为可以在客户端与网关之间的通信线路上加密以确保连接的安全。比如，**网关可以连接数据库，使用 SQL 语句查询数据**。另外，在 Web 购物网站上进行信用卡结算时，网关可以和信用卡结算系统联动 

### 利用网关可以由 HTTP 请求转化为其他协议通信 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/32.PNG)

**隧道**
隧道是在相隔甚远的客户端和服务器两者之间进行中转，并保持双方通信连接的应用程序 

隧道可按要求建立起一条与其他服务器的通信线路，届时使用 SSL 等加密手段进行通信。隧道的目的是确保客户端能与服务器进行安全的通信。隧道本身不会去解析 HTTP 请求。也就是说，请求保持原样中转给之后的服务器。隧道会在通信双方断开连接时结束 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/33.PNG)

### 35、保存资源的缓存(缓存服务器)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/34.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/35.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/36.PNG)

### 36、客户端的缓存 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/37.PNG)

### 37、HTTP 首部字段 

HTTP 首部字段是由首部字段名和字段值构成的，中间用冒号“:”分隔 

```http
首部字段名: 字段值
```

例如，在 HTTP 首部中以 Content-Type 这个字段来表示报文主体的对象类型 :

````http
Content-Type: text/html
````

其中“text”是主文档类型，“html”是子文档类型。“text/html”表示目标文档是text类型中的html文档

### 38、HTTP/1.1 通用首部字段 

通用首部字段是指，请求报文和响应报文双方都会使用的首部 

### 39、管理持久连接 

HTTP/1.1 版本的默认连接都是持久连接。为此，客户端会在持久连接上连续发送请求。当服务器端想明确断开连接时，则指定 Connection 首部字段的值为 Close 

```http
Connection: close
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/38.PNG)



**HTTP/1.1 之前的 HTTP 版本的默认连接都是非持久连接**。为此，**如果想在旧版本的 HTTP 协议上维持持续连接，则需要指定Connection 首部字段的值为 Keep-Alive** 

```http
Connection: Keep-Alive
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/39.PNG)

### 40、请求首部字段 

请求首部字段是从客户端往服务器端发送请求报文中所使用的字段，用于补充请求的附加信息、客户端信息、对响应内容相关的优先级等内容 

**Accept** 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/40.PNG)

Accept 首部字段可通知服务器，**用户代理能够处理的媒体类型及媒体类型的相对优先级**(若想要给显示的媒体类型增加优先级，则使用 q= 来额外表示权重值，**用分号（;）进行分隔类型和权重** 。1 为最大值。不指定权重 q 值时，默认权重为 q=1.0)。可使用 type/subtype 这种形式，一次指定多种媒体类型 当服务器提供多种内容时，将会首先返回权重值最高的媒体类型 

比如，如果浏览器不支持 PNG 图片的显示，那 Accept 就不指定image/png，而指定可处理的 image/gif 和 image/jpeg 等图片类型 

#### 几个媒体类型的例子 ：

**文本文件：**

```http
text/html, text/plain, text/css ...
application/xhtml+xml, application/xml ...
```

**图片文件 :**

```http
image/jpeg, image/gif, image/png ...
```

**视频文件 :**

```http
video/mpeg, video/quicktime ...
```

**应用程序使用的二进制文件 :**

```http
application/octet-stream, application/zip ...
```



**Host** 

虚拟主机运行在同一个 IP 上，因此使用首部字段 Host 加以区分 

例如；

```http
Host: www.hackr.jp
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/41.PNG)

首部字段 Host 会告知服务器，请求的资源所处的互联网主机名和端口号。**Host 首部字段在 HTTP/1.1 规范内是唯一一个必须被包含在请求内的首部字段** 

首部字段 Host 和以单台服务器分配多个域名的虚拟主机的工作机制有很密切的关联，这是首部字段 Host 必须存在的意义。请求被发送至服务器时，请求中的主机名会用 IP 地址直接替换解决。但如果这时，相同的 IP 地址下部署运行着多个域名，那么服务器就会无法理解究竟是哪个域名对应的请求。因此，就需要使用首部字段 Host 来明确指出请求的主机名。若服务器未设定主机名，那直接发送一个空值即可 

```http
Host:
```

### 41、HTTP 的缺点  

**通信使用明文（不加密），内容可能会被窃听**
**不验证通信方的身份，因此有可能遭遇伪装**
**无法证明报文的完整性，所以有可能已遭篡改** 

#### (不仅在 HTTP 上出现，其他未加密的协议中也会存在这类问题 )

由于 HTTP 本身不具备加密的功能，所以也无法做到对通信整体（使用 HTTP 协议通信的请求和响应的内容）进行加密。即，HTTP 报文使用明文（指未经过加密的报文）方式发送 

#### TCP/IP 是可能被窃听的网络 



### 加密处理防止被窃听 

**通信的加密** :

一种方式就是将通信加密。**HTTP 协议中没有加密机制，但可以通过和 SSL**（Secure Socket Layer，安全套接层）或 TLS（Transport Layer Security，安全层传输协议）**的组合使用，加密 HTTP 的通信内容** 

用 SSL 建立安全通信线路之后，就可以在这条线路上进行 HTTP 通信了。与 SSL 组合使用的 HTTP 被称为 HTTPS（HTTP Secure，超文本传输安全协议） 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/42.PNG)

**(传输层加密可以一直保留到目的地才解密，所以提供端对端的安全机制)**

**内容的加密：** 

还有一种将参与通信的内容本身加密的方式 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/43.PNG)

由于该方式不同于 SSL 或 TLS 将整个通信线路加密处理，所以内容仍有被篡改的风险 



**任何人都可发起请求** 

在 HTTP 协议通信时，由于不存在确认通信方的处理步骤，任何人都可以发起请求 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/44.PNG)

### 42、查明对手的证书 

虽然使用 HTTP 协议无法确定通信方，但如果使用 SSL 则可以。SSL 不仅提供加密处理，而且还使用了一种被称为证书的手段，可用于确定方 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/45.PNG)

### 43、HTTP+ 加密 + 认证 + 完整性保护 =HTTPS 

我们把添加了加密及认证机制的 HTTP 称为 HTTPS 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/46.PNG)

当浏览器访问 HTTPS 通信有效的 Web 网站时，浏览器的地址栏内会出现一个带锁的标记。对 HTTPS 的显示方式会因浏览器的不同而有所改变 

**HTTPS 并非是应用层的一种新协议**。只是 HTTP 通信接口部分用 SSL（Secure Socket Layer）和 TLS（Transport Layer Security）协议代替而已 

#### 通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和SSL 通信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/47.PNG)

SSL 是独立于 HTTP 的协议，所以不光是 HTTP 协议，其他运行在应用层的 SMTP 和 Telnet 等协议均可配合 SSL 协议使用 

#### HTTPS 也存在一些问题，那就是当使用 SSL 时，它的处理速度会变慢 (HTTPS 比 HTTP 要慢 2 到 100 倍 )

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/51.PNG)

SSL 的慢分两种。一种是指**通信慢**。另一种是指由于大量消耗CPU 及内存等资源，导致**处理速度变慢** 

### 44、加密和解密都会用到密钥 

使用两把密钥的公开密钥加密 

公开密钥加密使用一对非对称的密钥。一把叫做**私有密钥**（private key），另一把叫做**公开密钥**（public key）。顾名思义，私有密钥不能让其他任何人知道，而公开密钥则可以随意发布，任何人都可以获得 

使用公开密钥加密方式，发送密文的一方使用对方的公开密钥进行加密处理，对方收到被加密的信息后，再使用自己的私有密钥进行解密。利用这种方式，不需要发送用来解密的私有密钥，也不必担心密钥被攻击者窃听而盗走 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/48.PNG)

### HTTPS 采用混合加密机制 

因为公开密钥加密与共享密钥加密相比，其处理速度要慢，所以两者结合，利用各自优点

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/49.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/50.PNG)

### 45、以前的 HTTP 通信 

![](http://oklbfi1yj.bkt.clouddn.com/%E5%9B%BE%E8%A7%A3Http%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/52.PNG)



#### Ajax 的解决方法 

由于它只更新一部分页面，响应中传输的数据量会因此而减少 

Ajax 的核心技术是名为 XMLHttpRequest 的 API，**通过 JavaScript 脚本语言的调用就能和服务器进行 HTTP 通信** 

而利用 Ajax 实时地从服务器获取内容，有可能会导致大量请求产生。另外，Ajax 仍未解决 HTTP 协议本身存在的问题 

#### Comet 的解决方法 

一旦服务器端有内容更新了，Comet 不会让请求等待，而是直接给客户端返回响应。这是一种通过延迟应答，模拟实现服务器端向客户端推送（Server Push）的功能 

通常，服务器端接收到请求，在处理完毕后就会立即返回响应，但为了实现推送功能，Comet 会先将响应置于挂起状态，当服务器端有内容更新时，再返回该响应。因此，服务器端一旦有更新，就可以立即反馈给客户端 

#### 使用浏览器进行全双工通信的 WebSocket 

利用 Ajax 和 Comet 技术进行通信可以提升 Web 的浏览速度。但问题在于通信若使用 HTTP 协议，就无法彻底解决瓶颈问题。**WebSocket** 网络技术正是为解决这些问题而实现的一套**新协议及 API** 

WebSocket 技术主要是为了解决Ajax 和 Comet 里 XMLHttpRequest 附带的缺陷所引起的问题 

一旦 Web 服务器与客户端之间建立起 WebSocket 协议的通信连接，之后所有的通信都依靠这个专用协议进行。通信过程中可互相发送 JSON、XML、HTML 或图片等任意格式的数据**。由于是建立在 HTTP 基础上的协议，因此连接的发起方仍是客户端，而一旦确立 WebSocket 通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文** 

成功握手确立 WebSocket 连接之后，通信时不再使用 HTTP 的数据帧，而采用 WebSocket 独立的数据帧 



















