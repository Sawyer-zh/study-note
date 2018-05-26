---
title: 对MySQL数据表分区一定会提高查询效率吗
date: 2018-02-19 21:20:10
tags:
- MySQL
---

作为一个后端开发人员，我们一般都听说过**对数据表进行分区**的操作。甚至有开发人员说：这张表以后的数据量一定会很大，至少有好几千万条，我们对这张表分区吧。

然而，有时候并没有感受到查询速度的提升，**反而感觉更加的慢**了。原因就是不好的分区设计导致对磁盘的IO操作次数增加了。

我们现在来做个测试，数据有1000W条，储存引擎用的是InnoDB。

首先我们创建数据表：

```mysql
create table user (
    id int(11) not null auto_increment,
    key_id int(11) not null,
    primary key (id),
    key key_id (key_id)
) engine = innodb
partition by hash (id)
partitions 10;
```

我们采取的分区类型是hash分区，因为这样可以让数据比较均匀的储存在不同的分区中（我们之后可以看看数据分布情况）。且以主键id为标准进行分区。

`partitions 10`指的是对这张表分10个区。

好的，创建完数据表之后，我们插入1000W条数据。我们采用存储过程来实现：

```mysql
delimiter $$

create procedure myproc() 
begin 
declare num int; 
set num = 1; 
while num <= 10000000 do 
insert into user (key_id) values(num); set num = num + 1;
end while;
end
$$
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/1.png)

定义完了存储过程之后，我们调用它：

```mysql
call myproc()$$
```

执行存储过程大概有15分钟，耐心等待。

OK，接下来我们大致看看数据表中的数据：

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/2.png)

我们来查看下总共有多少条记录在数据表里面：

```mysql
select count(*) from user$$
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/3.png)

1000W条记录对吧，嗯，真刺激。

接下来我们来看看数据在这10个分区的分布情况：

```mysql
select table_name, partition_name, table_rows
from information_schema.partitions
where table_schema=DATABASE() AND table_name='user'\G
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/4.png)

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/5.png)

通过table_rows可以看出，记录比较均匀的分布在10个分区里面。和我们预期一致。

OK，接下来我们就开始查询比较，看看分区是否真的有用。我们先对主键进行查询：

```mysql
EXPLAIN PARTITIONS SELECT * FROM user WHERE id = 1\G
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/6.png)

可以看到，只查询了分区p1。因为一般B+树是2～3层，所以最多进行了2～3次IO操作。

这样吧，假设1000W条数据的单表构成的B+树有3层，而100W条数据表构成的B+树只有2层，那么，确实可以减少一次IO操作。但是如果100W和1000W行的数据构成的B+树层次都是一样的，可能都是2层。那么上述按照主键id分区的索引并不会带来性能的提高。

OK，分析到这里，我们再来看看可能让大家吃惊的一条查询：

```mysql
EXPLAIN PARTITIONS SELECT * FROM user WHERE key_id = 1\G
```

![](http://oklbfi1yj.bkt.clouddn.com/%E5%AF%B9MySQL%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%88%86%E5%8C%BA%E4%B8%80%E5%AE%9A%E4%BC%9A%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E6%95%88%E7%8E%87%E5%90%97/7.png)

可以看到，这一条查询对所有的分区（共10个）都进行了查询。这意味着什么？对磁盘的IO操作次数大大提高了。假设100W条记录构成的B+树只有2层好吧，那么对这10个分区进行查询也至少需要执行20次IO操作。很显然，因为分区后，以key_id列索引作为查询条件时，查询效率大大降低。