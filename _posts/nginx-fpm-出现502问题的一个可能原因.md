---
title: nginx(fpm)出现502问题的一个可能原因
date: 2018-01-03 16:05:09
tags:
- nginx
- fpm
---

为什么要重新配nginx呢？因为我一直以来都是用lnmp一键安装包，那时候看到配置就害怕。虽然省去了很多配置的问题，对新手比较友好，但是随着学习的深入，迟早是要分开来用的，不能老用一键安装包。碰巧今天测试秒杀系统的时候，模拟了5000个客户端去请求数据库，不知道怎么搞的就把MySQL搞坏了，也不知道怎么解决，索性卸了数据库。也把Nginx卸了，分开来安装......

我前几天在Windows上面分开来安装过。比较顺利，但是今天在Linux上面因为一个坑折腾了很久......

我简单说一下。首先看Nginx配置文件的关键部分：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/1.PNG)

我配置完了之后，写了一个简单的php脚本进行测试，打开网页，显示了nginx自带的error页面：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/2.PNG)

遇到这种问题怎么解决呢？我们不是复制里面的英文，然后谷歌、百度（这是我刚开始学习编程的做法）。我们一般是按下`F12`（我当前用的是UC浏览器，不同浏览器快捷键可能有所不同，谷歌也是`F12`），或者鼠标右键，选择检查，会弹出这个东西：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/3.PNG)

我们点击那个被我圈出来的英文：

然后再点击下面被我圈出来的东西：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/4.PNG)

得到如下图片：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/5.PNG)

我们会发现，在上面这张图片中有一个叫做：

```
Status Code:502 Bad Gateway (from disk cache)
```

的东西

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/6.PNG)

状态码为502，而且说网关出了问题。我们分析一下，这个网关可能是什么？我们知道PHP有一个SAPI叫做FPM，翻译过来就是`FastCGI进程管理`，那么CGI是什么呢？翻译过来就是通用网关接口（当初有一个技术官面试我的时候，让我解释了一下这个东西，小伙伴们一定要理解这个东西，这个东西关键，很多时候就是因为这个东西导致**网站**运行不起来）。

我们到了这一步，可以测试一下，HTML页面可以访问吗？

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/11.PNG)

OK，到这里，我们大致可以猜测一下是FPM某方面出了问题，缩小了出错的范围。

这个时候，我们就可以带着问题去谷歌和百度了。我百度出来的解决方案很多，大多数是什么FPM开的进程数量太少啊、资源耗尽了啊。这些配置我都有尝试过，但是都无法解决我的502问题。我也想：只有我一个人访问，怎么可能会是FPM开的进程数量太少、更不可能耗尽资源了。

我们继续回到第一张图片的Nginx的配置；

```
location ~ \.php$ {
            root           /home/wwwroot/default;
            fastcgi_pass  127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /home/wwwroot/default$fastcgi_script_name;
            include        fastcgi_params;
        }
```

我们来看一看哪些和FPM有关。第一个是网站的根目录，肯定是和FPM无关的。接下来是：

```
fastcgi_pass  127.0.0.1:9000;
```

这个东西是和FPM有关的，意思是告诉Nginx把客户端的请求发送给FPM，也就是找到FPM对吧。我想了一下，问题也是可能出在这里，因为Nginx找不到的话，也是无法把请求给FPM的。所以我去看了看FPM的配置，看看它监听哪个端口：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/7.PNG)

我们发现，它和Nginx的不一致，php这边监听的是sock文件然而nginx那边设置的是9000端口。我们把它修改一下：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/8.PNG)

```
listen = 127.0.0.1:9000
```

然后，问题就得到了解决：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/9.PNG)

然后下面这个地方也没有红色了：

![](http://oklbfi1yj.bkt.clouddn.com/nginx%28fpm%29%E5%87%BA%E7%8E%B0502%E9%97%AE%E9%A2%98%E7%9A%84%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0/10.PNG)

好的，这篇博客到这里差不多要结束了。相信大家也有收获吧。这篇博客不仅仅是教大家如何复制粘贴的去胡乱解决问题，而是与大家分享一下我一般是如何解决一个问题的。缩小范围，缩小谷歌、百度的范围，精确问题的所在，问题一般都可以解决。而且，大家也知道了FPM有**两种监听方式**、静态页面是不需要经过FPM的。

最后，在多说一句，基础知识很重要，如果你不知道FPM是什么，你不知道Nginx扮演什么角色，你不懂一点网络编程的知识，这个问题也够我们折腾了。

一个希望成为大牛的大三菜鸡......