# easyswoole使用笔记

## 1、开启

```shell
php easyswoole start
```

## 2、注意事项

- 修改了代码之后要记得重启服务器

- 不要在代码中执行sleep以及其他睡眠函数，这样会导致整个进程阻塞 exit/die是危险的，会导致worker进程退出
- Worker进程不得共用同一个Redis或MySQL等网络服务客户端，Redis/MySQL创建连接的相关代码可以放到onWorkerStart回调函数中
- 新手非常容易犯这个错误，由于easySwoole是常驻内存的，所以加载类/函数定义的文件后不会释放。因此引入类/函数的php文件时必须要使用include_once或require_once，否会发生cannot redeclare function/class 的致命错误


- **项目中类名称与类文件(文件夹)命名，均为大驼峰，变量与类方法为小驼峰**。
- 在HTTP响应中，于业务逻辑代码中echo $var 并不会将$var内容输出至相应内容中，请调用Response实例中的wirte()方法实现。

## 3、composer dumpautoload

