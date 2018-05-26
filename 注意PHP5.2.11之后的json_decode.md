# 注意PHP5.2.11之后的json_decode

**在PHP5.3以后,可以通过json_last_error()来验证转换是否正确**

因为字符串不是合法的json格式的串, 所以应该出错, 返回NULL。例如：

```
php -r "var_dump(json_decode('laruence'));"
//输出
NULL
```

但值得注意的是, 对于numeric_string, 都是返回numeric_string的数值形式：

```
php -r "var_dump(json_decode('0x3f34'));"
//输出
int(16180)
```







