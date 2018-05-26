# PHP实现文件上传和下载

### 1、文件上传原理

将客户端的文件上传到服务器端，再**将服务器端的临时文件移动到指定目录即可**

### 2、文件上传配置

#### 客户端

1、表单页面

2、表单的发送方式为**post**（不能使用**get**）

3、`<form>`里面添加`enctype="multipart/form-data"`

#### 服务端

使用`$_FILES`变量获取上传的文件

#### $_FILES:HTTP文件上传变量

##### $_FILES中保存这上传文件的信息

**name**

上传文件的名称

**type**

上传文件的MIME类型

**tmp_name**

上传到服务器上的临时文件名（我们之后操作的就是这个临时文件）

这个**tmp_name**保存的是一个绝对路径

**size**

上传文件的大小

**error**

上传文件的错误号

### 3、将服务器上的临时文件移动到指定目录下

```php
<?php
    
    $fileName = $_FILES['myFile']['name'];
    $type = $_FILES['myFile']['type'];
    $tmp_name = $_FILES['myFile']['tmp_name'];
    $error = $_FILES['myFile']['error'];
    $size = $_FILES['myFile']['size'];

    move_uploaded_file($tmp_name, 'uploads/'.$fileName);
```

注意：`move_uploaded_file`函数的第二个参数不能是一个目录名，一定要是一个文件名

### 4、文件上传配置

#### 服务器配置

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E5%92%8C%E4%B8%8B%E8%BD%BD/1.PNG)

其中：

如果`file_uploads`设置为**Off**，就不能上传文件了

`upload_tmp_dir`如果不进行设置，那么就会使用默认的

例如，在windows下面是：`C:\Windows\temp`；在Linux下面是`/tmp`

注意，传输的文件大小只要超过了`upload_max_filesieze`和`post_max_sieze`中任意一项允许的最大值都是不能上传成功的

### 5、错误信息说明

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E5%92%8C%E4%B8%8B%E8%BD%BD/2.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/PHP%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E5%92%8C%E4%B8%8B%E8%BD%BD/3.PNG)

### 6、文件的下载

```html
<!DOCTYPE html>
<html>
<head>
	<title>下载文件</title>
	<meta charset="utf-8">
</head>
<body>
<a href="/doDownload.php?filename=1.jpg">下载1.jpg</a>
</body>
</html>
```

**doDownload.php**文件：

```php
<?php

$fileName = $_GET['filename'];
var_dump($fileName);

//通过header()发送头信息，告诉浏览器如何来处理文件（通过附件的形式来处理）、处理哪个文件
header('content-disposition:attachment;filename='.$fileName);
//发送一个头信息，告诉浏览器文件的大小
header('content-length:'.filesize($fileName));
//读取文件的真正内容
readfile($fileName);
```

注意：如果没有这个`doDownload.php`脚本的话，当我们点击链接`<a href="1.jpg">1.jpg</a>`的时候，浏览器就会直接显示出图片，而不会下载图片

注意，那个请求头中的`filename='.$fileName`指的是下载下来的文件的文件名，不一定得和服务器上的那个文件名字一样（例如，我可以改为`filename=king_'.$fileName`）

使用`basename()`函数的原因是去除掉文件的路径，只保留文件名

注意，如果不使用`readfile()`函数的话，是获取不到文件的内容的