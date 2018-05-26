---
title: 用C++写一个服务器吧（二）
date: 2017-09-21 10:10:51
tags:
- C++
- Linux
- 网络编程
- 服务器开发
---

在这个系列的第一篇文章里面，我们说到了让浏览器来接收服务器发送给它的资源，并且显示出这个HTML页面。然后，结果并没有如预期的那样。在这篇文章，我们将解决这个问题。

好的，现在我们回忆一下第一篇文章。我们已经看到了，HTML文件里面的内容已经被我们完整的获取了。所以说，不是因为HTML的内容导致失败的。

然后，我们也看到了浏览器显示`malformed HTTP status code "/index.html"`这条信息。也就是说问题是出在了服务器那里。

我们再来审查元素看看：

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%BA%8C%EF%BC%89/5.PNG)

发现响应出问题了。

### 遵循协议

在这个系列的第一篇文章里，我们谈到了协议这部分内容。而协议顾名思义就是一套标准、一套规范。谁不遵循这个规范，我就不按照你的意思去做。

关于涉及到HTTP协议的那部分内容出现在了两个地方：

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%BA%8C%EF%BC%89/1.PNG)

和

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%BA%8C%EF%BC%89/2.2.PNG)

我们发现，第二张图片中的响应头信息是对的。只是少了一些具体的信息，我们把她完善一些：

```c++
sprintf(buff, "HTTP/1.0 200 OK\r\n");
sprintf(buff, "%sServer: HHTWS Web Server\r\n", buff);
sprintf(buff, "%sContent-length: %d\r\n", buff, file_size);
sprintf(buff, "%sContent-type: %s\r\n\r\n", buff, file_type);
```

然后我们看看第一张图片发送的信息。

因为它是从表示连接的`socket`文件描述符中读取信息的，因此，读到的自然也就是请求头的信息。而此时，我们又把请求头中的信息发送给浏览器，显然是不符合响应头的要求的。因此，我们把这一行删掉，来看看结果：

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%BA%8C%EF%BC%89/3.gif)

OK，美丽的HTML页面出现了！！

我们来审查一下元素，看看

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8C++%E5%86%99%E4%B8%80%E4%B8%AAWeb%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%A7%EF%BC%88%E4%BA%8C%EF%BC%89/4.PNG)

这些不正是我们在代码里面写的响应头吗？

OK。接下来做些小小的优化。

### 优化

虽然`write()`不太可能返回一个**部分写**的结果。而且，对`write`系统调用来说没有`EOF`情况。对于普通文件，除非发生一个错误，否则`write`将保证写入所有的请求。

所以，对于普通文件，不需要进行循环写入了。然后，**对于其他类型--例如套接字--大概得有个循环来保证你真的写入了所有请求的字节**

```c++
int makesure_write(int connect_fd, void* buff, int n) {
	int remain_num = n;
    int success_write_num = 0;
    char* buff_current_postition = (char*)buff;

    while (remain_num > 0) {
    	success_write_num = write(connect_fd, buff_current_postition, remain_num);
    	remain_num -= success_write_num;
    	buff_current_postition += success_write_num;
    }

    return n - remain_num;
}
```

好了，这次的教程结束。happy ending

