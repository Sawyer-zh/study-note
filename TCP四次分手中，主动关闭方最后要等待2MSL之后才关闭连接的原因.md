# TCP四次分手中，主动关闭方最后要等待2MSL之后才关闭连接的原因

和TCP三次同步握手不一样的是，TCP关闭连接用四次握手来实现，即A--->B Fin, B--->A ACK, B--->A Fin, A--->B ACK，为什么要这样？

A--->B Fin, B--->A ACK ，A属于主动关闭方，收到B的ACK后，A到B的方向连接关闭，即half shutown ，这时A不能再发送数据了

这种状态下B还是可以单向发送数据的，B的数据发送完毕，也做关闭动作了：

B--->A Fin, A--->B ACK

B收到ACK，关闭连接。但是**A无法知道ACK是否已经到达B，于是开始等待**？等待什么呢？假如ACK没有到达B，B会为FIN这个消息超时重传 timeout retransmit ，那如果A等待时间足够，又收到FIN消息，说明ACK没有到达B，于是再发送ACK，直到在足够的时间内没有收到FIN，说明ACK成功到达。这个等待时间至少是：B的timeout + FIN的传输时间，为了保证可靠，采用更加保守的等待时间2MSL

MSL，Maximum Segment Life，这是TCP 对TCP Segment 生存时间的限制

TTL， Time To Live ，IP对IP Datagram 生存时间的限制，255 秒，所以 MSL一般 = TTL = 255秒

**A发出ACK，等待ACK到达对方的超时时间 MSL，等待FIN的超时重传，也是MSL**，所以如果2MSL时间内没有收到FIN，说明对方安全收到FIN