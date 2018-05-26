# MySQL锁机制

首先我认为在谈到数据库各种机制前（尤其是MySQL这种插件式plugin存储引擎的数据库），你一定要声明你使用的存储引擎，不然问题无意义，此处我假定你的第1、2条问题是基于innoDB存储引擎下的，并且我的理论也是基于innoDB引擎。
明确概念，锁分为两种： 
读锁->共享锁 （S）
写锁 -> 排它锁 （X）
写操作的为排它锁，读操作的为排它锁，
理论上来说只要被操作对象（例如 id =1这行）出现排它锁，那么除了该事务可以操作这行，其余事务全部无权对这行进行任何操作，无论是读操作还是写操作

对了，selct ... for update并不属于SQL规范，所以实际上应该避免使用。
select ... for update 是进行排它锁操作的，但是这条语句在事务中才有意义，因为在非事务情况下使用，一下子就完成了锁定与解锁的操作，无任何实际意义。
还有一点很重要的是innoDB在隔离级别为REPEATABLE READ和READ COMMITTED的事务操作中，并没有直接使用行级锁，而是使用了乐观机制MVCC，即大多数的读操作都不用加锁，即使是写锁也只是锁必要的行[《了解MVCC》](http://www.veitor.net/article/436.html)

回到你的问题：
1、“SELECT username,passwd FROM g_test LIMIT 1 FOR UPDATE” 是否加锁了？

> 是的，加了一个行级排它锁。

2、“for update写加锁是否对delete语句有效？”

> 加上for update是一个写锁（即排它锁），它对delete当然有效（delete无法操作），但还是上面的观点，在REPEATABLE READ和READ COMMITTED的事务操作中，使用了MVCC，所以你可以惊奇的发现即使用了for update这条排它锁语句后，其他事务依旧可以进行读操作。

3、“MyISAM加锁是全表锁吗？”

> 对的，MyISAM存储引擎只有表级锁（table-level locking）

4、“innoDB是行级锁？”

> 不全对，innoDB具有行级锁和表级锁，但它默认是行级锁。

5、“事务和锁是否有超时机制”

> 事务本身是没有超时机制的，但是如果事务是由于锁表而造成的超时原因，是会进行回滚操作。innodb默认锁超时时间为50秒，通过设置变量innodb_lock_wait_timeout来实现。































