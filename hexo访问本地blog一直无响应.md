# hexo使用localhsot:4000访问本地blog一直无响应

Hexo 3.0 把服务器独立成了个别模块，您必须先安装 hexo-server 才能使用。

```
$ npm install hexo-server --save

```

安装完成后，输入以下命令以启动服务器，您的网站会在 [http://localhost](http://localhost/):4000 下启动。在服务器启动期间，Hexo 会监视文件变动并自动更新，您无须重启服务器。

```
$ hexo server

```

如果您想要更改端口，或是在执行时遇到了 EADDRINUSE 错误，可以在执行时使用 -p 选项指定其他端口，如下：

```
$ hexo server -p 5000
```