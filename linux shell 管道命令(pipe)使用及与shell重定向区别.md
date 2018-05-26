# linux shell 管道命令(pipe)使用及与shell重定向区别

**管道命令**操作符是：”|”,它**仅能处理经由前面一个指令传出的正确输出信息，也就是 standard output 的信息，对于 stdandard error 信息没有直接处理能力**。然后，传递给下一个命令，作为标准的输入 standard input

![](http://oklbfi1yj.bkt.clouddn.com/linux%20shell%20%E7%AE%A1%E9%81%93%E5%91%BD%E4%BB%A4%28pipe%29%E4%BD%BF%E7%94%A8%E5%8F%8A%E4%B8%8Eshell%E9%87%8D%E5%AE%9A%E5%90%91%E5%8C%BA%E5%88%AB/1.PNG)

command1正确输出，作为command2的输入 然后comand2的输出作为，comand3的输入 ，**comand3输出就会直接显示在屏幕上面了**



**注意：**

**1、通过管道之后：comand1,comand2的正确输出不显示在屏幕上面**

**2、管道命令只处理前一个命令正确输出，不处理错误输出**

**3、管道命令右边命令，必须能够接收标准输入流命令才行(否则传递过程中数据会抛弃)**

**4、"|"管道两边都必须是shell命令**

举个例子：

```
$ cat test.sh | grep -n 'echo'
5:    echo "very good!";
7:    echo "good!";
9:    echo "pass!";
11:    echo "no pass!";
#读出test.sh文件内容，通过管道转发给grep 作为输入内容
```

```
$ cat test.sh test1.sh | grep -n 'echo'
cat: test1.sh: 没有那个文件或目录
5:    echo "very good!";
7:    echo "good!";
9:    echo "pass!";
11:    echo "no pass!";
#cat test1.sh不存在，错误输出打印到屏幕，正确输出通过管道发送给grep 
```



### 管道命令与重定向区别

1、左边的命令应该有标准输出 | 右边的命令应该接受标准输入

2、左边的命令应该有标准输出 > 右边只能是文件

3、左边的命令应该需要标准输入 < 右边只能是文件

4、管道**触发两个子进程**执行"|"两边的程序；而重定向是在**一个进程内**执行





