# ThinkPHP框架中，在第二重volist的eq里面使用第一重循环中的key变量的方法

去掉eq两边的大括号｛｝

因为在eq标签里面，thinkphp的模板变量会被解析

而在普通的html标签里面就不会被解析了，所以要加上大括号｛｝

