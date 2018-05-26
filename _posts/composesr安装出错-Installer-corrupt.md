---
title: 'composesr安装出错:Installer corrupt'
date: 2017-07-08 20:16:30
tags:
- php
- composer
---

这个问题的原因是https协议

**解决方法：**

# 1、先下载`composer-setup.php`在你当前的目录

在终端输入：`php -r "copy('http://getcomposer.org/installer', 'composer-setup.php');"`

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/1.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/2.PNG)

# 2、把composer安装到你的bin路径下(这样你可以全局使用它)

在终端输入：`php composer-setup.php --install-dir=/usr/local/bin --filename=composer`

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/3.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/4.PNG)

# 3、取消那个composer的链接

在终端输入：`php -r "unlink('composer-setup.php');"`

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/5.PNG)

此时`composer-setup.php`这个文件已经没了

# 4、检查composer是否安装成功

在终端输入：`composer -v`

![](http://oklbfi1yj.bkt.clouddn.com/composesr%E5%AE%89%E8%A3%85%E5%87%BA%E9%94%99:Installer%20corrupt/6.PNG)

### 好了，开启你的composer之旅吧