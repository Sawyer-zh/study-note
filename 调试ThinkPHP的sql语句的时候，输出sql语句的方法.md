# 调试ThinkPHP的sql语句的时候，输出sql语句的方法

举个例子：

```php
M("info_object_focus",Null,'DB_CORE')->where("unfocus_time = Null AND object_id = {$infoObject['id']}")->field('time, user_id, unfocus_time')->buildSql();
```

在语句的末尾执行buildSql()函数之后，它不会去去执行这条sql语句，而是会打印出这条sql语句，这时候，我们把这条sql语句放在数据库管理系统里面去调试它