# SHELL编程笔记

所有的脚本都要以`#!/bin/bash`开头

如果要执行脚本要赋予 755 权限：`chmod 755 脚本名`

### 1、Shell可以用来简化系统管理操作

​	例如我要添加100个用户，如果是手动来执行命令来添加，估计会累死。所以这时候如果用脚本来实现就更方便了

### 2、Bash/Shell变量

bash是linux的标准shell，我们说的**bash和shell都是指的一个东西**。所以我们说的shell编程实际上就是指的linux编程或者bash编程

shell在定义变量的时候不需要加`$`符号，在调用变量的时候需要加上`$`符号

#### 变量命名规则

变量名是不能以数字开头的

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/1.PNG)

此时，它会把`5x=6`这一整条赋值语句当成一条linux命令

怎么记忆呢？例如：

```shell
123=456
```

这明显是不符合人的思考习惯的

**注意：等号两边是不能加空格的**

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/2.PNG)

因为，linux命令是采取输入命令名字然后加上空格的形式。所以，如果左边加了空格的话，会把变量名当作linux命令

同理，右边也不行：

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/3.PNG)

并且我们发现，**因为`jie`和`cao`的中间有空格，所以需要使用引号**括起来(如果引号里面的值没有特殊含义，那么使用单引号或双引号都可以；如果有例如`$`这样的，使用双引号的话就会解析它，使用单引号就不解析)

#### 变量按照存储数据分类

字符串型、整型、数组、浮点型、日期型等等。**但是，这些分类对于linux来说都不起作用，都默认是字符串型**

#### shell变量分类

​	1、用户自定义变量(本地变量)

​	2、环境变量

​		这种变量中主要保存的是和系统操作环境相关的数据。环境变量也可以自定义，但是对系统生效的环境变量名和变量作用是固定的

​	3、位置参数变量

​		这种变量主要是用来向脚本当中传递参数或数据的，变量名**不能自定义**，变量**作用是固定的**

​		位置参数变量就是**预定义变量的一种**

​	4、预定义变量

​		是Bash中已经定义好的变量，变量名**不能自定义**，变量**作用也是固定的**

​		不能自己去新增加预定义变量。这种变量完全是系统确定好的

#### 变量的默认类型

字符串型(所以不能直接进行加减乘除，要先进行转换)。例如：

```shell
x=5
y=6
z=$x+$y
echo $z
```

此时输出：

```shell
5+6
```

而不是我们以为的整数11

#### 变量叠加

```shell
x=124
x="$x"456
echo $x
```

输出：

```shell
123456
```

#### 查看所有变量

```shell
set
```

使用`set -u`命令，则会在使用没有定义的变量的时候报错`unbound variable`

#### 查看环境变量

```shell
env
```

可以查看到用户自定义的环境变量和系统默认的环境变量

#### 删除变量

```shell
unset name
```

因为删除变量并不是要删除这个变量的值，所以不需要加`$`符号(如果加了就达不到删除变量的效果)

### 3、环境变量

#### 环境变量与用户自定义变量的区别

用户自定义变量只有在当前的Shell中生效(局部变量)

环境变量在当前Shell和这个Shell的所有子Shell中生效(全局变量)

#### 进入子shell

输入`bash`命令可以进入当前`shell`的子`shell`：

```shell
bash
```

我们可以通过以下命令来查看(进程树)：

```shell
pstree
```

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/4.PNG)

我们可以看到这里有两个bash，其中左边的bash是父shell，右边的bash是子shell

也就是说，我们通过远程命令连接(从图中的sshd守护进程可以看出)，在父shell中执行了bash进入了子shell，然后在子shell中执行了pstree命令

#### 退出子shell

```shell
exit
```

#### 用户自定义环境变量

```shell
export 变量名=变量值
```

或

```shell
变量名=变量值
```

```shell
export 变量名
```

在父shell中定义的变量不能在子shell中删除

建议用户自定义的环境变量写成大写，因为Linux中的命令都是小写，把环境变量写成大写就不会和Linux命令产生冲突了

#### 常用系统默认的环境变量

`HOSTNAME`：主机名

`SHELL`：当前的shell

`TERM`：终端环境

`HISTSIZE`：历史命令条数

`SSH_CLIENT`：当前操作环境是用ssh连接的，这里记录客户端ip(注意，这里的SSH_CLIENT和我们在百度中搜ip得到的值是一样的)

`SSH_TTY`：ssh连接的终端时`pts/1`

`USER`：当前登录的用户

##### PATH环境变量

它的值是用冒号分割开的目录：

```shell
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

这些目录是**系统搜索命令的路径**

**在Linux中，执行可执行文件的常用方法是：用绝对路径(或者相对路径)去找到这个命令，然后按下回车**。但是，我们在实际操作过程中出来没有这么操作过。例如，我们直接使用`ls`就可以列出目录，而不需要使用`/bin/ls`。原因就是这个`$PATH`起了作用。当我敲入一个命令，它会在PATH中定义的这些路径里面一个一个去找，直到找到

而且**当我们按下`tab`键，它去`PATH`路径里面搜索，然后自动补全这些命令或这文件名**

##### 增加PATH变量的值

```
PATH="$PATH":/root/otherPath
```

这句话实际上是使用了变量叠加(可以理解为字符串拼接)。即，双引号先把`$PATH`变量解析，然后再和字符串`:/root/otherPath`连接起来

##### PS1环境变量

**PS1变量**

命令提示符设置

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/7.PNG)

例如：

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/8.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/9.PNG)



普通用户的提示符是`$`，超级用户的提示符是`#`

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/5.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/6.PNG)

#### 语系变量

##### 当前语系查询

```shell
locale
```

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/10.PNG)

可以看到，一些语系被赋值给了一些变量

其中，en_US是**美式英文语系**的意思，例如：

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/11.PNG)

可以看到，红线划出的部分是英文显示

而如果是中文语系，则会会显示中文：

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/12.PNG)

**这些变量当中，最主要生效的就是变量：`LANG`、`LC_ALL`**

`LANG`：定义系统主语系的变量

`LC_ALL`：定义整体语系的变量

这两个变量中当前生效的变量是`LANG`

##### 查看系统当前语系

```shell
echo $LANG
```

##### 查看Linux支持的所有语系

```shell
locale -a | more
```

#### 系统默认语系

##### 查询系统默认语系

```shell
cat /etc/sysconfig/i18n
```

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/13.PNG)

系统默认语系和前面所说的语系变量是有所不同的。系统默认语系是放在一个文件里面的(无论当前生效的语系是什么，只要下次一开机，立即生效的就是这个文件里面所保存的系统默认语系)

**linux一切皆文件，放在文件里的才是永久生效的**

##### Linux中文支持

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/14.PNG)

对于第二点，指的是：例如我们使用xshell这类的远程工具，只要我们设置xshell的选项为中文语系即可(注意，安装操作系统的时候要选择支持中文语系)

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/15.PNG)

### 4、位置参数变量

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/16.PNG)

举个例子：

```shell
#!/bin/bash

sum=$(( $1 + $2 ))
echo $sum
```

文件名为`code1.sh`。此时，在终端输入：

```shell
./code1.sh 10 20
```

其中的`./code1.sh`代表`$0`

得到结果：

```shell
30
```

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/17.PNG)

举个例子：

```shell
#!/bin/bash

echo "\$* can shu shi : $*"
echo "\$@ can shu ye shi : $@"
echo "\$# can shu ge shu shi : $#"
```

文件名为`code2.sh`。此时，在终端输入：

```shell
./code2.sh 1 2 3 4 5
```

得到结果：

```shell
$* can shu shi : 1 2 3 4 5
$@ can shu ye shi : 1 2 3 4 5
$# can shu ge shu shi : 5
```

#### `$*`和`$@`区别

举个例子：

```shell
#!/bin/bash

for x in "$*"
	do
		echo $x
	done

for y in "$@"
	do
		echo $y
	done
```

文件名为`code3.sh`。此时，在终端输入：

```shell
./code3.sh 1 2 3 4
```

得到结果：

```shell
1 2 3 4
1
2
3
4
```

说明`$*`的`1 2 3 4`是一个整体，所以循环只执行一次；而`$@`的`1 2 3 4`四个，所以循环执行四次

### 5、预定义变量

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/18.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/19.PNG)



#### 接收键盘输入

![](http://oklbfi1yj.bkt.clouddn.com/SHELL%E7%BC%96%E7%A8%8B%E7%AC%94%E8%AE%B0/20.PNG)

例如：

```shell
#!/bin/bash

read -p "please input your name: " name
echo $name
```

文件名为`code4.sh`。此时，在终端输入：

```shell
./code4.sh
```

### 6、在脚本中输出换行

```shell
echo -e "\n"
```

注意：`-e`不能漏了，否则不能识别`\n`































