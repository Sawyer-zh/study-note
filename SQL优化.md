# SQL优化

### 1、通过show status 命令了解各种SQL的执行频率

例如：我想显示所有开头是`Com_`的参数的信息：

```
show status like 'Com_%';
```

我们通常比较关心的几个参数是：

**Com_select**：执行SELECT操作的次数，一次查询只累加1

**Com_insert**：执行INSERT操作的次数，对于批量插入的INSERT操作，只累加一次

**Com_update**：执行UPDATE操作的次数

**Com_delete**：执行DELETE操作的次数

通过这几个参数，我们可以很容易的了解当前数据库的应用是以插入更新为主还是以查询操作为主

此外，一下几个参数便于用户了解数据库的基本情况

Connections：试图连接MySQL服务器的次数

Uptime：服务器工作时间

Slow_queries：慢查询的次数

### 2、通过EXPLAIN分析低效SQL的执行计划

使用方法很简单，在 `explain`的后面加上那条你想要执行的SQL语句就行了



