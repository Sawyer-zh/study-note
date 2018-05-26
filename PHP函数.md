# PHP函数

### 1、boolval

把值转化为对应的`bool`值

### 2、mysql_query("SET NAMES utf8"); 

有时候，取数据库中的数据，然后显示在浏览器的数据全是 ??????? 问号乱码，这是因为**数据的编码格式和浏览器的编码格式不匹配**导致。此时，可以在取数据之前进行一次编码设置

例如，数据库中存放的数据使用UTF8编码

```shell
create table tablename 
( 
    id int not null auto_increment,
    title varchar(20) not null,
    contnet varchar(300) defalut null,
    primary key ('id')
)engine=MyISAM DEFAULT CHARSET=UTF8; 
```

然后，在插入数据之前执行: 

```php
mysql_query("SET NAMES utf8");
```

然后 

```php
mysql_query("insert into tablename .....");
```

取数据之前执行：

```php
mysql_query("SET NAMES utf8");
```

然后

```php
mysql_query("select * from tablename");
```

注意：此处读出的编码是把原来编码的内容重新经过编码后输出的,比如输出内容所在页面是GBK编码,那么在读出的时候在页面显示也为乱码,所以在查询之前执行 mysql_query("SET NAMES gbk"),在页面就可以正常显示GBK编码的文字内容 









