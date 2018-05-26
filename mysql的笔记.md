# mysql的一些操作

### 1、清空一个表可以使用：

​	truncate table tbname;

执行这个操作后，id将会从1开始计数



### 2、获得数据表最后一条记录的方法：

​	select * from table order by id DESC limit 1;

也就是先把所有记录倒序排，然后只获得一条



### 3、修改字段的方法：

​	 ALTER TABLE table_name CHANGE old_field_name new_field_name field_type;

注意，如果不想改变字段的名字，那么old_field_name 和 new_field_name两个值应该相同



### 4、增加一个新的字段的方法：

​	 ALTER TABLE table_name ADD field_name field_type;



### 5、查看创建表的时候的创建语句的方法：

​	show create table tbname;



### 6、在创建数据表的时候，最后一行的逗号不能有



### 7、导入数据表的方法

记得得先创建一个空的数据库



### 8、使用多表查询的联结的时候，**一般在`on`的后面写两张表的关联条件，而在`where`的后面写查询条件**，例如：

```mysql
SELECT * FROM class,school WHERE class.school_id = school.id AND class.`name` = '一年级（2）班' AND school.`school_name` = '育讯通实验小学a'
```



### 9、建表的语法：

```mysql
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

如果想要显示中文的话，在建表的时候记得指定字符集为：`utf8`



### 10、连接数据库的方法：

```
mysql -h127.0.0.1 -P3306 -uroot -proot
```

注意，这里的`-h127.0.0.1`指的是连接的是本地的数据库(如果要连接远程服务器的数据库，那么修改这个ip即可)



### 11、交叉连接

笛卡尔乘积通俗的说，就是两个集合中的每一个成员，都与对方集合中的任意一个成员有关联。可以想象，在SQL查询中，如果对两张表join查询而没有join条件时，就会产生笛卡尔乘积

交叉联接返回左表中的所有行，左表中的每一行与右表中的所有行组合。交叉联接也称作笛卡尔积

















