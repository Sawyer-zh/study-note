# 超清晰的 DNS 原理入门指南

## DNS服务器

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/1.jpg)

DNS服务器的IP地址，有可能是动态的，每次上网时由网关分配，这叫做DHCP机制；也有可能是事先指定的固定地址。Linux系统里面，DNS服务器的IP地址保存在`/etc/resolv.conf`文件。

上例的DNS服务器是192.168.1.253，这是一个内网地址。有一些公网的DNS服务器，也可以使用，其中最有名的就是Google的8.8.8.8和Level 3的4.2.2.2。

## 域名的层级

DNS服务器怎么会知道每个域名的IP地址呢？答案是分级查询。

请仔细看前面的例子，每个域名的尾部都多了一个点。

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/2.jpg)

比如，域名http://math.stackexchange.com显示为math.stackexchange.com.。这不是疏忽，而是所有域名的尾部，实际上都有一个根域名。

举例来说，http://www.example.com真正的域名是www.example.com.root，简写为www.example.com.。因为，根域名`.root`对于所有域名都是一样的，所以平时是省略的。

根域名的下一级，叫做”**顶级域名**”（top-level domain，缩写为TLD），比如.com、.net；再下一级叫做”**次级域名**”（second-level domain，缩写为SLD），比如http://www.example.com里面的.example，**这一级域名是用户可以注册的**；再下一级是**主机名**（host），比如http://www.example.com里面的www，又称为”三级域名”，这是用户在自己的域里面为服务器分配的名称，是**用户可以任意分配的**。

总结一下，域名的层级结构如下。

```
主机名.次级域名.顶级域名.根域名
# 即
host.sld.tld.root
```

## 根域名服务器

DNS服务器根据域名的层级，进行分级查询。

需要明确的是，**每一级域名都有自己的NS记录，NS记录指向该级域名的域名服务器。这些服务器知道下一级域名的各种记录**。

**所谓”分级查询”，就是从根域名开始，依次查询每一级域名的NS记录，直到查到最终的IP地址**，过程大致如下。

1. 从”根域名服务器”查到”顶级域名服务器”的NS记录和A记录（IP地址）
2. 从”顶级域名服务器”查到”次级域名服务器”的NS记录和A记录（IP地址）
3. 从”次级域名服务器”查出”主机名”的IP地址

仔细看上面的过程，你可能发现了，没有提到DNS服务器怎么知道”根域名服务器”的IP地址。回答是**”根域名服务器”的NS记录和IP地址一般是不会变化的，所以内置在DNS服务器里面**。

## 分级查询的实例

dig命令的+trace参数可以显示DNS的整个分级查询过程。

```
dig +trace math.stackexchange.com
```

上面命令的第一段列出根域名.的所有NS记录，即所有根域名服务器。

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/3.jpg)

根据**内置的**（也就是说这些根域名服务器的IP地址是大家都知道的，不需要去问根域名服务器的父亲节点，实际上也不存在根域名服务器的父亲节点）根域名服务器IP地址。**DNS服务器向所有这些IP地址发出查询请求**，**询问**[http://math.stackexchange.com](https://link.zhihu.com/?target=http%3A//math.stackexchange.com)的**顶级域名服务器com.的NS记录**。**最先回复的根域名服务器将被缓存，以后只向这台服务器发请求**。

接着是第二段。

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/4.jpg)

上面结果显示.com域名的13条NS记录，同时返回的还有每一条记录对应的IP地址。

然后，DNS服务器向这些顶级域名服务器发出查询请求，询问[http://math.stackexchange.com](https://link.zhihu.com/?target=http%3A//math.stackexchange.com)的次级域名[http://stackexchange.com](https://link.zhihu.com/?target=http%3A//stackexchange.com)的NS记录。

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/5.jpg)

上面结果显示[http://stackexchange.com](https://link.zhihu.com/?target=http%3A//stackexchange.com)有四条NS记录，同时返回的还有每一条NS记录对应的IP地址。

然后，DNS服务器向上面这四台NS服务器查询[http://math.stackexchange.com](https://link.zhihu.com/?target=http%3A//math.stackexchange.com)的主机名。

![](http://oklbfi1yj.bkt.clouddn.com/%E8%B6%85%E6%B8%85%E6%99%B0%E7%9A%84%20DNS%20%E5%8E%9F%E7%90%86%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/6.jpg)

上面结果显示，[http://math.stackexchange.com](https://link.zhihu.com/?target=http%3A//math.stackexchange.com)**有4条A记录，即这四个IP地址都可以访问到网站**。并且还显示，最先返回结果的NS服务器是[http://ns-463.awsdns-57.com](https://link.zhihu.com/?target=http%3A//ns-463.awsdns-57.com)，IP地址为205.251.193.207。

## DNS的记录类型

**域名与IP之间的对应关系，称为”记录”（record）**。根据使用场景，”记录”可以分成不同的类型（type），前面已经看到了有A记录和NS记录。

常见的DNS记录类型如下。

（1） A：地址记录（Address），**返回域名指向的IP地址**。

（2） NS：域名服务器记录（Name Server），返回保存**下一级域名信息的服务器地址**。该记录只能设置为域名，不能设置为IP地址。

（3）MX：邮件记录（Mail eXchange），返回接收电子邮件的服务器地址。

（4）CNAME：规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转，详见下文。

（5）PTR：逆向查询记录（Pointer Record），只用于从IP地址查询域名，详见下文。





