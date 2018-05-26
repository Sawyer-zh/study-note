# 高性能的PHP API接口开发

一个接口的功能尽量单调

## 1、API基础知识讲解

### 模板模式开发与API模式开发的区别

模板模式开发：

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/1.PNG)

模板处理环节就是序号4这个环节，也就是说把变量给模板，然后将模板信息拼接成一个输出。通过序号5的过程返回给Web服务器。即模板的处理是在服务器完成的。

API模式开发：

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/2.PNG)

可以看出API的模式工作更加的精简、清晰。我们是做数据拼装和处理。而传统的模式我们不仅要做数据的拼装和处理，还要做模板的处理。

### Restful API

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/3.PNG)

## 2、用户类接口实现

### 数据表设计

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/4.PNG)

#### MyISAM / InnoDB 

MyISAM 表锁范围是全表锁（也是一个缺点，因为只要某个进程写一张表<哪怕只是一条记录>，那么其他进程就不可以对这张表操作），而InnoDB是行级锁。

MyISAM支持全文检索，InnoDB适用于对表操作很频繁的表。

### API接口实现

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/5.PNG)

#### 用户类接口的实现

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/6.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/7.png)

在单机的情况下，php有将session存储在内存缓存中的能力，只要改php.ini就可以。但是在如今的互联网环境下，单机可以承载的session的容量是非常有限的，而且我们的webserver可能部署在多台机器。此时，我们可以利用图中的session Store。现在有用redis来做session store，使用php和redis来做session store网上的方案也很多。

##### 如何防止机器恶意注册与机器模拟用户登录

常用的方式是验证码

##### 如何实现密码找回功能

一般利用**邮箱和手机号**

#### 文章类接口的实现

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/8.png)

## 3、扩展第三方能力

![](http://oklbfi1yj.bkt.clouddn.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84PHP%20API%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91/9.png)

### Push服务

给客户端推送消息













