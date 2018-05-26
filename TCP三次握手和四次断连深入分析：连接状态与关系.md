# TCP三次握手和四次断连深入分析：连接状态与关系

## 连接过程总结

1、只有当server端listen之后，client端调用connect才能成功，否则就会返回RST响应拒绝连接

2、只有当accept后，client和server才能调用recv和send等io操作

