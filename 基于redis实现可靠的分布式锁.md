# 基于redis实现可靠的分布式锁

## 分布式锁解决什么问题

分布式锁解决的是 **多台机器 – 多个进程** 之间的同步问题，因为不同的机器之间互斥锁、信号量等锁机制无法使用。

## redis分布式锁

构造函数：

```php
function __construct(array $servers, $retryDelay = 200, $retryCount = 3) {
    $this->servers = $servers;
    $this->retryDelay = $retryDelay;
    $this->retryCount = $retryCount;
    $this->quorum  = min(count($servers), (count($servers) / 2 + 1));
}
```

- 需要传入的是redis的若干master节点地址，并且这些master是纯内存模式且无slave的。
- retryDelay是设置每一轮lock失败或者异常，多久之后重新尝试下一轮lock。
- retryCount是指最多几轮lock失败后彻底放弃。
- quorum体现了分布式里一个有名的”鸽巢原理”，也就是如果大于半数的节点操作成功则认为整个集群是操作成功的；在这里的意思是，如果超过1/2的(>=N/2+1)redis master调用锁成功，则认为获得了整个redis集群的锁，假设A用户获得了集群的锁，那么接下来的B用户只能获得<=1/2的redis master的锁，相当于无法获得集群的锁。

初始化redis连接：

```Php
private function initInstances() {
    if (empty($this->instances)) {
        foreach ($this->servers as $server) {
            list($host, $port, $timeout) = $server;
            $redis = new \Redis();
            $redis->connect($host, $port, $timeout);
            $this->instances[] = $redis;
        }
    }
}
```

- **遍历每个redis master，建立到它们的连接并保存起来**；
- 因为需要用到”鸽巢原理”，也就是redis数量足够产生”大多数”这个目的：因此redis master数量最好>=3台，因为2台的话大多数是2台(2/2+1)，这样任何1台故障就无法产生”大多数”，那么整个分布式锁就不可用了。

请求1个redis上锁：

```php
private function lockInstance($instance, $resource, $token, $ttl) {
    return $instance->set($resource, $token, ['NX', 'PX' => $ttl]);
}
```

- 请求某一台redis，如果key=resource不存在就设置value=token（算法生成，全局唯一），并且redis会在ttl时间后自动删除这个key

请求1个redis放锁：

```Php
private function unlockInstance($instance, $resource, $token) {
    $script = '
            if redis.call("GET", KEYS[1]) == ARGV[1] then
                return redis.call("DEL", KEYS[1])
            else
                return 0
            end
        ';
    return $instance->eval($script, [$resource, $token], 1);
}
```

- 请求某一台redis，给它发送一段lua脚本，如果resource的value不等于lock时设置的token则说明锁已被它人占用无需释放，否则说明是自己上的锁可以DEL删除。
- lua脚本在redis里原子执行，在这里即保障GET和DEL的原子性。

请求集群锁：

```Php
public function lock($resource, $ttl)
{
    $this->initInstances();
    $token = uniqid();
    $retry = $this->retryCount;
    do {
        $n = 0;
        $startTime = microtime(true) * 1000;
        foreach ($this->instances as $instance) {
            if ($this->lockInstance($instance, $resource, $token, $ttl)) {
                $n++;
            }
        }
        # Add 2 milliseconds to the drift to account for Redis expires
        # precision, which is 1 millisecond, plus 1 millisecond min drift
        # for small TTLs.
        $drift = ($ttl * $this->clockDriftFactor) + 2;
        $validityTime = $ttl - (microtime(true) * 1000 - $startTime) - $drift;
        if ($n >= $this->quorum && $validityTime > 0) {
            return [
                'validity' => $validityTime,
                'resource' => $resource,
                'token'    => $token,
            ];
        } else {
              foreach ($this->instances as $instance) {
                  $this->unlockInstance($instance, $resource, $token);
              }
        }
        // Wait a random delay before to retry
        $delay = mt_rand(floor($this->retryDelay / 2), $this->retryDelay);
        usleep($delay * 1000);
        $retry--;
    } while ($retry > 0);
  
    return false;
}
```

- 首先整个lock过程最多会重试retry次，因此外层有do while。
- 为了获取”大多数”的锁，因此遍历每个redis master去lock，统计成功的次数。
- 因为遍历redis master进行逐个上锁需要花费一定的时间，因此在第1个redis上锁前记录时间T1，结束最后一个redis上锁动作的时间点T2，此时第1个redis的TTL已经消逝了T2-T1这么长的时间。
- 为了保障在锁内计算期间锁不会失效，我们剩余可以占用锁的时间实际上是TTL – (T2 – T1)，因为越靠前上锁的redis其剩余时间越少，最少的就是第1个redis了。
- drift值用于补偿不同机器时钟的精度差异，怎么理解呢：
  - 在我们的程序看来时间过去了(T2-T1)，剩余的锁时间认为是TTL-(T2-T1)，在接下来的剩余时间内进行计算应该不会超过锁的有效期。
  - 但是第1台redis机器的机器时钟也许跑的比较快（比如时钟多前进了1毫秒），那么数据会提前1毫秒淘汰，然而我们认为TTL-(T2-T1)秒内锁有效，而redis相当于TTL-(T2-T1)-1秒内锁有效，这可能导致我们在锁外计算。（drift+1）
  - 另外，我们计算(T2-T1)之后到返回给lock的调用者之间还有一段代码在运行，这段代码的花费也将占用一些时间，所以drift应该也考虑这个。（drift+1）
  - 最后，ttl * 0.01的意思是ttl越长，那么时钟可能差异越大，所以这里做了一个动态计算的补偿，比如ttl=100ms，那么就补偿1ms的时钟误差，尽量避免遇到锁已过期而我们仍旧在计算的情况发生。
- 如果锁redis成功的次数>1/2，并且整个遍历redis+锁定的过程的耗时 没有超过锁的有效期，那么lock成功，将剩余的锁时间（TTL减去上锁花费的时间）+ 锁的标识token 返回给用户。
- 如果上锁中途失败（返回key已存在）或者异常（不知道操作结果），那么都认为上锁失败；如果上锁失败的数量超过1/2，那么本次上锁失败，需要遍历所有redis进行回滚（回滚失败也没有办法，其他人只能等待我们的key过期，并不会有什么错误）。

释放集群锁：

```Php
public function unlock(array $lock) {
    $this->initInstances();
    $resource = $lock['resource'];
    $token    = $lock['token'];
    foreach ($this->instances as $instance) {
        $this->unlockInstance($instance, $resource, $token);
    }
}
```

- 遍历所有redis，利用lua脚本原子的安全的释放自己建立的锁。





























