# Redis从入门到高可用

## 1、Redis特性

### 多种数据结构

除了早先的字符串、列表、哈希、集合、有序集合，最新的版本还提供了位图(BitMaps)、HyperLogLog：超小内存唯一值计数、GEO：地理信息定位。

### 主从复制

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/1.png)

主服务器的数据可以同步到从服务器上面。

### 高可用、分布式

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/2.png)

## 2、Redis 3种启动方法

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/3.png)

### 最简启动

```shell
./redis-server
```

### 动态参数启动

```shell
./redis-server --port 6380
```

### 配置文件启动

```shell
 ./redis-server configPath
```

## 3、Redis 常用配置

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/4.png)

一般我们会创建一个`config`目录，里面放了redis服务器多个配置文件，每个配置文件对应一个redis服务器。

查看redis配置文件的方法：

```shell
cat redis-6381.conf | grep -v "#" | grep -v "^$"
```

其中参数`-v`代表过滤掉的意思。所以会过滤掉`#`和空格。

执行上面的命令，得到如下结果：

```shell
bind 127.0.0.1
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
注意：后面还省略了一些
```

然后，我们可以把去掉注释的配置文件重定向另一个配置文件，达到复制的效果：

```
cat redis-6381.conf | grep -v "#" | grep -v "^$" > redis-6382.conf
```

基本配置：

假设这是redis-6382.conf配置文件：

```shell
bind 127.0.0.1
daemonize yes
port 6382
dir "/downloads/redis-4.0.8/data"
logfile "6382.log"
```

（其中，logfile是与dir相对应的，也就是说，`6382.log`是在目录`/downloads/redis-4.0.8/data`下的）

启动：

```shell
./redis-server config/redis-6382.conf
```

因为是以守护进程的方式启动的，所以在终端上面看不到启动成功的画面，可以使用命令：

```shell
ps -ef | grep redis-server | grep 6382
```

## 4、API的理解和使用

### 通用命令

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/5.png)

#### keys

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/6.png)

keys命令一般不在生产环境使用，因为它的时间复杂度是O(n)。

##### keys *有什么用

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/7.png)

#### dbsize

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/8.png)

dbsize可以在生产环境中使用，因为它的时间复杂度是O(1)

#### exists

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/9.png)

exists可以在生产环境中使用，因为它的时间复杂度是O(1)

#### del

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/10.png)

#### expire、ttl、persist

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/11.png)

ttl的结果如果是 -1，代表key存在，并且没有过期。

#### type

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/12.png)

### 时间复杂度

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/13.png)

## 5、数据结构和内部编码

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/14.png)

用空间换时间的话，可以使用压缩列表结构。比如元素个数比较小的时候。

### 字符串类型

#### 使用场景

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/18.png)

#### 命令

##### get、set、del

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/19.png)

##### incr、decr、incrby、decrby

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/20.png)

##### set、setnx、set xx

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/24.png)

setxx可以理解为更新操作（因此需要key存在才可以更新）

##### mget、mset

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/25.png)

使用一次mget和n次get的区别：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/26.png)

所以关键是多出了n次网络时间。

#### 实战

1、记录网站每个用户个人主页的访问量。

例如：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/21.png)

单线程无竞争，意思是Redis是天然适合做计数器的，所以在并发执行incr的时候，不会发生竞争问题。

2、缓存视频的基本信息（视频的数据源在MySQL中）

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/22.png)

伪代码：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/23.png)

### 哈希

#### 哈希键值结构

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/27.png)

#### 特点

Mapmap（即map的map）

#### 命令

都是h开头的

##### hget、hset、hdel

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/28.png)

##### hexists、hlen

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/29.png)

##### hmget、hmset

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/30.png)

### 列表

#### 列表结构

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/31.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/32.png)

#### 命令

##### lrem

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/33.png)

#### 使用场景

##### TimeLine

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/34.png)

### 集合

#### 集合内使用场景

1、抽奖系统

2、Like、赞、踩

3、标签

#### 集合间使用场景

1、共同关注

### redisObject

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/15.png)

## 6、redis 单线程为什么那么快

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/16.png)

### 单线程需要注意的问题

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/17.png)

## 7、慢查询

### 生命周期

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/35.png)

客户端请求Redis服务器的一个完整生命周期：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/36.png)

因为Redis是单线程的，所以第二个步骤需要排队。

两点说明：

- 慢查询是发生在第3阶段
- 客户端超时（即客户端很久都没有得到Redis的响应结果）不一定慢查询，但慢查询是客户端超时的一个可能

### 两个配置

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/37.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/38.png)

### 配置方法

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/39.png)

### 慢查询命令

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/40.png)

### 运维经验

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/41.png)

## 8、piepline

### 流水线作用

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/42.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/43.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/44.png)

### 使用建议

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/45.png)

## 9、发布订阅

### 角色

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/46.png)

### 模型

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/47.png)

这种模型类似于生产者与消费者模型。

每个订阅者是可以订阅多个频道的：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/48.png)

注意一个问题，加入一个发布者已经发布了一条消息到一个频道，一个新的订阅者去订阅这个频道，它是收不到之前的消息的。

### API

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/49.png)

### 消息队列和Redis的发布订阅区别

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/50.png)

发布订阅是发布者发布一条消息，所有的订阅者都可以收到消息。而消息队列是一个抢的功能，也就是说，只有一个消息订阅者可以收到（Redis没有去实现消息队列这样的功能）。

## 10、位图

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/51.png)

例子：

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/52.png)

每个字节是字母对应的ASCII值。

通过位图，我们可以取到每一个位对应的值。

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/53.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/54.png)

### 命令

#### setbit

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/55.png)

## 11、HyperLogLog

基于HyperLogLog算法：用极小空间完成独立数量统计。

它的**本质还是字符串**。

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/56.png)

### 使用例子

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/57.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/58.png)

### 内存消耗

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/59.png)

### 局限性

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/60.png)

因为HyperLogLog只能够用来统计数据量，无法取出单条数据。

而且如果我们需要精确统计的话，用HyperLogLog可能不太好，因为它很难得到精确的答案。

## 12、GEO

GEI(地理信息定位)：存储经纬度，计算两地距离，范围计算等。

### 使用例子

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/61.png)

#### geoadd

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/62.png)

#### geopos

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/63.png)

#### geodist

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/64.png)

#### georadius

可以算出指定范围内的member。

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/65.png)

### 应用场景

1、类似微信摇一摇的功能。

2、外卖定位的功能。

## 13、Redis持久化的取舍和选择

### 什么是持久化

redis所有数据保持在内存中，对数据的更新将**异步**保持在磁盘上。

### 持久化的方式

#### 快照

假如说现在数据库里有10条数据，那我相当于拍一张照片，把数据拷贝出来，作为一个快照。即**某时某点数据的一个备份**。

例如，MySQL的Dump、Redis的RBD就是这种方式。

### 写日志

就是说，我们的数据库只要做任何数据的更新，我们就将它记录在日志当中。当某时候，我们需要将数据进行一个恢复的时候，我们只需要把这个日志拿过来，**重走一遍完整的过程**，就可以获取完整数据的一个变化。

例如，MySQL的 Binlog、Redis的 AOF。

### RDB

#### 什么是RDB

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/66.png)

#### 主要的3种触发方式

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/67.png)

#### save命令

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/68.png)

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/70.png)

##### 存在的问题

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/69.png)

它是阻塞的。

#### bgsave

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/71.png)

#### save与bgsave对比

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/72.png)

#### 自动生成RDB

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/73.png)

图中代表的含义是，如果在60秒内，对数据进行了10000次改变，那么会自动生成RDB文件；如果在300秒内，对数据进行了10次改变，那么会自动生成RDB文件；如果在900秒内，对数据进行了1次改变，那么会自动生成RDB文件。

**一般我们不会去配置自动生成RDB**。

#### 触发生成RDB文件的机制

- 全量复制
- debug reload
- shutdown

### AOF

#### RDB的缺点

- 耗时、耗性能
- 不可控、丢失数据

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/74.png)

因为生成RDB文件的本质是往硬盘中写数据，所以非常耗时。

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/75.png)

如果在T4时间点Redis服务器发生了宕机，那么在T3时间点写入的数据，就会丢失。（但是在T1时间点写入的数据，还是会被保留下来）

#### AOF运行原理--创建

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/76.png)

每次写入一条命令，就会在AOF文件里面追加一条日志（实际上会先写在缓冲区当中）。

#### AOF运行原理--恢复

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/77.png)

#### AOF的3种策略

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/78.png)

##### always

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/79.png)

##### everysec

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/80.png)

##### no

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/81.png)

这种方式(no 策略)是由操作系统决定什么时候把数据写入AOF文件中。

##### 三种策略的对比

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/82.png)

#### AOF重写

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/83.png)

##### 作用

- 减少磁盘占用量
- 加速恢复速度

##### 实现方式

###### bgrewriteafo命令

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/84.png)

这种方式会创建一个子进程，然后子进程去完成AOF重写的操作。

###### AOF重写配置

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/85.png)

##### 重写流程

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/86.png)

### RDB和AOF的抉择

#### RDB和AOF比较

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/87.png)

其中，启动优先级的意思是当Redis服务器启动的时候，既有RDB文件又有AOF文件的时候，优先加载AOF文件；体积小指的是RDB文件下，因为会**对数据进行压缩**；轻重指的是产生RDB文件这个过程是很重的，因为需要把数据完整的写入到磁盘中（比如，第二次产生RDB文件需要从头开始）。而产生AOF文件是一个追加的过程（也就是说，不需要从头开始产生）。

#### RDB最佳策略

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/88.png)

#### AOF最佳策略

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/89.png)

#### 最佳策略

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/90.png)

## 14、开发运维常见问题

### fork操作

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/91.png)

#### 改善fork

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/92.png)

#### 子进程开销和优化

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/93.png)

### 硬盘优化

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/94.png)

## 15、Redis复制的原理和优化

### 单机

单机的意思是在一台机器上去部署一个Redis节点。

### 主从复制

#### 作用

主从复制是实现**高可用**的方式。

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/95.png)

图片的意思是右边的Redis节点（从）去复制左边的Redis节点（主）的内容。

- 数据副本
- 扩展读性能

#### 一主多从

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/96.png)

可以实现**流量的分流、负载均衡、读写分离（主节点负责写，然后从从节点读出数据）**

注意，**一个从只能有一个主，而一个主可以有多个从**。

#### 例子

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/97.png)

左边的主Redis节点做出的修改，会在右边的从Redis节点反映出来。

#### 简单总结

- 一个master可以有多个slave
- 一个slave只能有一个master
- 数据流向是单向的，master到slave

#### 主从复制的方式

##### slaveof命令

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/98.png)

这条命令的意思是让redis-6380成为redis-6379的从，也就是说，redis-6379是主，而redis-6380是从。

###### 取消主从复制之间的关系

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/99.png)

##### 修改配置

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/100.png)

其中：

```Redis
slave-read-only yes
```

的意思是只能对这个从节点进行读的操作。这样做是为了保证主从之间数据一致，以防从节点写入了数据，而主节点没有这个数据。

##### 两种方式比较

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/101.png)

#### 查看主从关系的命令

```shell
info replication
```

#### 需要注意的地方

当把一个redis节点作为另一个redis节点的从时候，这个从原来的数据会被全部清空，以保证从的数据与主的数据一致。

### 全量复制和部分复制

#### 全量复制

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/102.png)

也就是说，从节点**通过主节点传过来的RDB文件和buffer的数据**，来实现对主节点数据的全量复制。

##### 开销

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/103.png)

#### 部分复制

![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/104.png)

### 可能遇到的问题

#### 读写分离

 ![](http://oklbfi1yj.bkt.clouddn.com/Redis%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E9%AB%98%E5%8F%AF%E7%94%A8/105.png)









































