## TCP中断可以用3次挥手吗？

首先，TCP要记住一点，非常关键，记住了这点繁复兀杂世界将变得简单而有条理：**SYN 、Data、FIN 这三者发出去，对方一定要ACK！**

很多人有一个误区，即认为TCP连接是通信的全部(实际上还要看application)，其实并不是这样，让我们来复习一下TCP连接断开的过程

假定TCP client端主动发起断开连接

1 client端的application 接受用户断开TCP连接请求，这个是由用户触发的请求，以消息的方式到达client TCP

2 client TCP 发送 FIN=1 给 server 端 TCP

3 server 端TCP 接收到FIN=1 断开连接请求，需要咨询 客户端的application的意见，所以需要发消息给客户端的application，消息内容：对方(指的是用户)要断开连接，请问您老人家(指的是客户端的application)还有数据要发送吗？如果有数据请告知，没有数据也请告知！然后就是等待客户端的application 的回应。既然是等待客户端的application的回复，为何不早点把对client FIN 的ACK发出去呢(因为，服务器接收到了客户端要断开client --> server 方向的TCP连接，无论客户端的application是否要发数据，服务器都可以发送一个确认这个断开的信号。因为，客户端的application如果不发数据，因为服务器发送了确认信号，所以可以直接断开；如果客户端的application发送数据，因为服务器发送了确认信号，所以，当客户端的application发送完了数据之后，就可以断开了)？ 事实上TCP也是这么做的，收到对方的断开连接请求，立马发ACK予以确认，client --> server 方向连接断开

![](http://oklbfi1yj.bkt.clouddn.com/Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/7.PNG)

这就是为什么报文段5会存在的原因(因为服务端需要询问客户端的application是否还有数据要发送给服务端)

实际上报文段5的回应是同意了断开 client --> server 的TCP连接。因为，如果可能会有半连接状态，也就是服务端不打算和客户端断开连接。如果此时没有了报文段5，则服务端就不会回复客户端一个ack

4.1 如果 server端有数据需要发送，则继续发送数据，一直到数据发送完毕。然后 服务器的application 发close消息给TCP，现在可以关闭连接，然后Server TCP 发FIN=1 断开 server -->client方向的连接(因为 client-->server 的TCP连接已经关闭了，所以，client不能够咨询服务器的application是否还有数据要发送)

4.2 如果 server端没有数据发送，application回应close消息给TCP，现在可以关闭连接，然后Server TCP 发FIN=1 断开 server -->client方向的连接(因为 client-->server 的TCP连接已经关闭了，所以，client不能够咨询服务器的application是否还有数据要发送)

### (注意：这篇文章里面的application不一定都是指客户端，因为，服务器也可以是某个application)

TCP连接是虚拟连接，通过双向的消息、消息确认来模拟物理连接