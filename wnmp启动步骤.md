# wnmp启动步骤

## nginx

先配置nginx：

```
location / {
            root   D:\wwwroot;
            index  index.html index.htm index.php;
        }
```

```
location ~ \.php$ {
            root           D:\wwwroot;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  D:\wwwroot$fastcgi_script_name;
            include        fastcgi_params;
        }
```

首先，先启动nginx：

```shell
start nginx
```

如果nginx再运行的过程中，修改了nginx的配置，那么需要重新启动nginx。

## php

然后再启动php-cgi：

```shell
./php-cgi -b 127.0.0.1:9000 php.ini
```

## mysql

如果是mac，并且是通过brew安装的mysql，那么可以通过

```
mysql.server start
```

来启动。

可能root用户启动回失败，需要安装mysql服务器的用户来启动。（注明：root用户不能用brew）



如果报错：

```
ERROR! Multiple MySQL running but PID file could not be found (13841 13941 )
```

可以先查处mysql的进程id：

```
ps aux | grep mysql
```

然后再杀掉进程，再来使用`mysql.server start`启动mysql。

