# PHP命令行用法总结

### 1、执行某个php脚本文件

```
php 文件名
```

### 2、执行一段PHP代码(不需要在`<?php  ?>`里面执行)

```shell
php -r "代码"
```

例如：

```shell
php -r "var_dump('huanghantao');"
```

### 3、启动PHP内置的服务器

进入项目文档的根目录，然后执行：

```shell
php -S localhost:4000
#此命令会启动一个PHP Web服务器，地址是 localhost。这个服务器监听的端口是4000。当前工作目录是这个Web服务器的文档根目录
```







