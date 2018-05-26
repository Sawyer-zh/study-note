# PHP实现get/post请求中的注意点

1、multipart/form-data这种编码格式,特别是需要通过form表单上传文件的时候,一定需要该编码格式。这种编码格式是**将需要发送的数据以控件为单位进行分割处理**,然后**添加到http请求中的request body中**

2、get请求是将发送的数据转化成key=value的形式，然后进行urlencode编码（处理例如 + 之类的特殊字符），然后添加到url中。用到的编码格式为：`application/x-www-form-urlencoded编码格式`

3、post请求可以使用`application/x-www-form-urlencoded编码格式`或`multipart/form-data`，如果采用`application/x-www-form-urlencoded编码格式`的格式，也是将需要发送的数据处理成key=value格式的字符串,然后添加到http请求中的request body中

4、还经常使用post发送json类型的数据,发送json类型的数据我们需要采用`application/json`这种编码格式

















