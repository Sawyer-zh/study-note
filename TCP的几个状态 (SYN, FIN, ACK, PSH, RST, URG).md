## TCP的几个状态 (SYN, FIN, ACK, PSH, RST, URG)

对于我们日常的分析有用的就是前面的五个字段：SYN, FIN, ACK, PSH, RST, URG

它们的含义是：

SYN表示建立连接

FIN表示关闭连接

ACK表示响应

PSH表示有 DATA数据传输

RST表示连接重置

其中，ACK是可能与SYN，FIN等同时使用的，比如SYN和ACK可能同时为1，它表示的就是建立连接之后的响应，

如果只是单个的一个SYN，它表示的只是建立连接

TCP的几次握手就是通过这样的ACK表现出来的。

但SYN与FIN是不会同时为1的，因为前者表示的是建立连接，而后者表示的是断开连接。

RST一般是在FIN之后才会出现为1的情况，表示的是连接重置