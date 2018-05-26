### Socket常用几种类型

 Socket是一组编程接口（API）, 是对TCP/IP协议的封装和应用。介于传输层和应用层,大致驻留在 OSI 模型的会话层，向应用层提供统一的编程接口。应用层不必了解TCP/IP协议细节。直接通过对Socket接口函数的调用完成数据在IP网络的传输

![](http://oklbfi1yj.bkt.clouddn.com/Socket%E5%B8%B8%E7%94%A8%E5%87%A0%E7%A7%8D%E7%B1%BB%E5%9E%8B/1.PNG)

**基于传输层差异，4种类型的Socket:**
   

(1)**基于TCP的Socket**:

​		提供给应用层可靠的流式数据服务，使用TCP的Socket应用程序协议：BGP，HTTP，FTP，TELNET等。优点：基于数据传输的可靠性

**(2)基于UDP的Socket:**

​		适用于数据传输可靠性要求不高的场合。基于UDP的Socket应用程序或协议有：RIP，SNMP，L2TP等

**(3)基于RawIp的Socket:**

​		非连接，不可靠的数据传输。特点：**能使应用程序直接访问网络层**。基于RawIp的Socket有**ping** ,tracert,ospf等

**(4)基于链路层的Socket**

​		为IS-IS协议提供的Socket接口。使IS-IS协议可通过Socket直接访问链路层。非连接，不可靠通信服务





