# 在windows的命令行下执行php代码的方法

首先找到含有php.exe的那个目录，例如，我的php.exe在    D:\wamp\bin\php\php5.5.12里

那么，我就把这个路径添加到环境变量的系统变量的path里面（注意用分号隔开）

然后，在window的搜索框里搜索cmd

之后再输入php  -v

如果显示了你所用的php版本号，那么配置环境变量成功



此时，你就可以在输入php这个命令来执行php代码了

例如：

我进入了一个叫做php-code的目录里面，里面有一个叫做code1.php的php文件

此时，我可以在命令行中键入php   code1.php来执行它