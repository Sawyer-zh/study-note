# mysql 中的分区

把一张表按照某种规则`（range/list/hash/key等）`分成多个区域（页/文件）保存

**同一个分区表中的所有分区必须是同一个存储引擎**

**注意：无论哪种分区，要么分区表上没有主键/唯一键，要么分区键必须有一个是主键/唯一键**

### **1.range分区**

range分区是利用取值范围（区间）划分分区，区间要连续并且不能互相重叠，使用`values less than`操作符进行分区定义

**例一：**

```mysql
CREATE TABLE `test`.`partition_t2`(  
  `id` INT UNSIGNED NOT NULL,
  `username` VARCHAR(30) NOT NULL,
  `email` VARCHAR(30) NOT NULL,
  `birth_date` DATE NOT NULL
) ENGINE=MYISAM
PARTITION BY RANGE(id)(
   PARTITION t21 VALUES LESS THAN (10),
   PARTITION t22 VALUES LESS THAN (20),
   PARTITION t23 VALUES LESS THAN MAXVALUE
);
```

上例中定义了一个包含3个分区（t21,t22,t23）的range分区表，`这个有点类似与高级语言中的switch语句`。解释如下：当id<10的时候，在t21分区；当20>id>=10的时候，在t22分区；当id>=20时候，在t23分区(注意最大值的写法 -- **MAXVALUE**)

### **2.list分区**

list分区创建**离散的值**列表（**类似mysql中的enum类型数据**）来划分分区，使用`values in`操作符来分区。list分区**不必要声明任何特定的顺序**。list有很多方面类似于range

```mysql
CREATE TABLE `test`.`partition_t4`(  
  `id` INT UNSIGNED NOT NULL,
  `username` VARCHAR(30) NOT NULL,
  `email` VARCHAR(30) NOT NULL,
  `birth_date` DATE NOT NULL
) ENGINE=MYISAM
PARTITION BY LIST(id)(
   PARTITION t41 VALUES IN (1,2),
   PARTITION t42 VALUES IN (3,6),
   PARTITION t43 VALUES IN (5,4),
   PARTITION t44 VALUES IN (7,8)
);
```

上面的例子是，当id为1或2，在t41分区；当id为3或6，在t42分区，以此类推...

### **3.hash分区**

hash分区主要用来分散热点读取，确保数据在预定确定个数分区中尽可能的平均分布。一个表执行hash分区，mysql会对分区键应用一个散列函数，以此确定数据应该放在n个分区中的哪一个分区。hash分区支持两种散列函数（分区方式）：`取模算法（默认hash分区方式）`和`线性的2的幂的运算法则（liner hash 分区）`

#### 取模算法（默认hash分区方式）的例子：

```mysql
#创建一个5个hash分区的myisam表
CREATE TABLE `test`.`partition_t1`(  
  `id` INT UNSIGNED NOT NULL,
  `username` VARCHAR(30) NOT NULL,
  `email` VARCHAR(30) NOT NULL,
  `birth_date` DATE NOT NULL
) ENGINE=MYISAM
PARTITION BY HASH(MONTH(birth_date))
PARTITIONS 5;
```

**mysql不推荐使用涉及多列的hash表达式**

**常规hash在分区管理上带来的代价太大了，不适合灵活变动的分区的需求**



#### 线性hash分区的例子：

```mysql
CREATE TABLE `test`.`partition_t5`(  
`id` INT UNSIGNED NOT NULL,
`username` VARCHAR(30) NOT NULL,
`email` VARCHAR(30) NOT NULL,
`birth_date` DATE NOT NULL
) ENGINE=MYISAM
PARTITION BY LINEAR HASH(id)
PARTITIONS 5;
```

上例中，创建了一个5个分区的线性hash分区

- 线性hash分区优点：在分区维护上，mysql能够处理更加迅速
- 线性hash分区缺点：各个分区之间数据分布不太均衡

**hash分区允许用户自定义的表达式**

**hash分区只支持整数分区**

**要指明分区键**

### **4.key分区**

**key分区不允许使用用户自定义的表达式**

**key分区支持除了blob或text类型之外的其他数据类型分区**

**创建key分区表的时候，可以不指定分区键，默认会选择使用主键/唯一键作为分区键，没有主键/唯一键,必须指定分区键**(**自己指定分区键的时候，分区键不一定得是主键/唯一键**)

```mysql
CREATE TABLE `test`.`partition_t6`(  
  `id` INT UNSIGNED NOT NULL,
  `username` VARCHAR(30) NOT NULL,
  `email` VARCHAR(30) NOT NULL,
  `birth_date` DATE NOT NULL
) ENGINE=MYISAM
PARTITION BY LINEAR KEY(email)
PARTITIONS 5;
```

