# 使用MySQL时候出现的一些问题汇总

1、Access denied for user 'root'@'localhost' (using password: YES)

​	这是没有连接上你的MySQL数据库的意思，你需要先检查一下你登录MySQL的用户名和密码是否正确

2、在执行判断null值的时候，应该用 `is null`而不能用`= null`，例如：

```mysql
SELECT `time`,`user_id`,`unfocus_time` FROM `info_object_focus` WHERE ( unfocus_time = Null AND object_id = 16 )
```

这条语句是查询不出来结果的，但是不会报错，正确的写法应该是：

```mysql
SELECT `time`,`user_id`,`unfocus_time` FROM `info_object_focus` WHERE ( unfocus_time IS Null AND object_id = 16 )
```

如果某个字段的值为null，**那么该字段直接进行 = 或者 != 运算，返回值都只有一个，就是false**

3、在建表的时候，如果某个字段使用了VARCHAR属性，那么需要指明它的宽度，否则会报错，而INT类型的就不用