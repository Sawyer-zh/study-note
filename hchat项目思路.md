# hchat项目思路

项目以iphone6这款手机的标准来开发



数据暂时都是模拟的，json格式，命名为data.json



路由部分：

1、消息messages -> 

2、联系人contactPerson

3、动态dynamic

这三个部分都使用了better-scroller插件



其中，底部的东西是起一个触发路由的作用，写成一个底部组件footer，在根组件里面来使用



进入页面首先显示的是message组件的部分，消息组件里面分为：

1、头部组件header，里面有头像，消息字样的标题和一个加号的标志

2、搜索框组件

3、消息列表，里面是一些好友发给我们的消息



联系人contactPerson组件里面分为：

1、头部组件header，里面有头像，联系人字样的标题和一个两个字

2、搜索框组件

3、新朋友区块

4、一个用来路由的区块，可以分别路由到好友、群、多人聊天、设备公众号

5、好友分组区块有特别关心、么么哒、家人、同学、学长学姐、不认识这几个默认的分组，他们都可以点击然后展开



