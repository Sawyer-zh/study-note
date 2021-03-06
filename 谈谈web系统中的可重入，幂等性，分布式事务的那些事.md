## 谈谈web系统中的可重入，幂等性，分布式事务的那些事

- 可重入：在并发访问下，仍旧可以保证正确性。
- 幂等性：**相同的程序输入（重复输入多次），获得相同的程序输出**（或者说产生相同的影响）。
- 分布式事务：与单机事务相对，跨越网络，多实例，**保证N件事情都做或者都不做**。

**可重入**可以通过加互斥锁解决，或者只用单线程，通过串行化访问绕开并发，正确性就得到保证。

如果代码运行在很多服务器上，机器经常性的宕机，网络偶尔会堵塞或者中断，那么又如何保证服务能够正常运行呢？

假设实现一个交易订单系统，那么订单通常会经历几个基本状态：未付款，过期未支付，已付款，已使用，退款中，已退款。基于这个背景，分别看看有什么办法解决上述问题，让程序继续正确的运行下去。

## 可重入

因为订单(order)保存在mysql中，许多php程序会并发的去修改订单，最简单的办法就是在mysql中锁住订单所在的行记录，也就是在一个transaction中执行select for update，但是并不建议使用大量的慢事务和行锁，对innodb带来了很多锁管理成本和并发能力限制，所以不建议这样做。

**mysql行锁是一种悲观锁**，**只能阻塞排队逐个修改**。其实**在解决可重入问题时，我们一般采用乐观锁来防止并发引起的数据更新错误**，以付款场景为例，先来看看问题本质：

1. 微信支付系统调用我们的接口，告知我们收到了order订单付款。
2. 从mysql读取这个order。
3. 如果order的status是”待付款”，那么调用mysql更新order为”已付款”。
4. 如果order的status是”过期未支付”，那么拒绝微信支付的这笔付款，原路退回给用户。

如果步骤3发现order为”待付款”之后暂停几秒，在同一时刻有一个crontab定时任务通过扫描mysql发现订单超过30分钟没有支付，于是将order状态改为了”过期未支付”。此后步骤3继续执行，又将订单改为了”已付款”，那么相当于用户绕过了我们的支付时间限制，成功进行了付款，这是我们不允许的。

如果我们在步骤2中以及crontab任务中均通过select for update锁住这一个order，那么就不可能出现上述的并发场景了。

如果我们采用乐观锁思路，步骤2不加锁，将步骤3的更新语句改为update order_table set status=”已付款” where order_id=xxx and status=”待付款”, 其中红色部分是新增部分，通过增加这个判定，在上面的并发场景下，mysql的更新将不生效，因为update执行之前crontab已经将mysql中的status变更为”过期未支付”了。

此时，我们的代码逻辑发现update没有生效，那么就可以认为有其他并发更新的插入，所以理论上应当重新查询order获取最新状态重新处理，此时发现order的status已经变更为”过期未支付”，最终会进入步骤4，将这笔付款原路打回。实际上，为了代码的逻辑的清晰性与架构合理性，通常并不会立即重查订单，而是抛出一个异常结果给上游（微信支付），由上游（微信支付）稍后重新调用我们，从而触发新一轮的完整逻辑。

## 幂等性

其实接着上面的例子说，幂等性也不是很难理解。当微信支付通知我们用户付款的时候，期望我们回复2个明确的结果：接受付款 Or 打回付款。

我们知道，既然微信和我们是跨网络交互的，必然存在异常，比如：请求被处理但是应答丢失了，请求没被处理，请求超时因为下游处理很慢，等等。一旦通讯异常那么这次调用是否对下游产生了影响，以及影响是什么，都是未知的，这种场景下只能进行重试。既然重试必然存在，那么当发生重试的时候，我们的程序能否始终保持一致的应答就是幂等性的典型体现。在这里，我们的应答要么总是接收付款，要么总是打回退款，不可以摇摆。

还是拿付款阶段来说，举2个典型的场景。第一种，我们成功完成了1-3步骤返回”接受付款”，但是微信没有收到应答，于是发起重试。第二种，我们在步骤3乐观锁更新失败抛出异常给微信，微信没有收到正确应答，于是发起重试。

第一种场景下，微信来重试的时候，我们查询order的status必然已经是”已付款”，此时我们要做的应该是依旧应答微信”接受付款”，这是幂等性典型的体现。

第二种场景下，微信来重试的时候，我们查询order的status已经是”过期未支付”，此时我们要做的是应答”拒绝付款”，因为首次微信调用我们抛出异常也就是没有明确答复，所以我们依旧遵循了应答的前后一致性，无论接下来微信重试多少次均会”拒绝付款”。

这里再说说红色部分”请求超时因为下游处理很慢”，这里可能触发一个场景就是第一次请求的php还没有处理完成，重试的请求又抵达了，也就是”可重入”问题也一起掺和进来了。当我们同时面临”可重入”,”幂等性”两个问题的时候，程序能否按照我们的预期继续执行呢？答案：可以正确执行。

如果读者此前没有相关的经验心得，可以先枚举一下各种时序，看一下分别更新了什么，返回了什么，这个场景不算复杂。我这里举1个时序，其他的可以自己设想一下。old-php读取到”待付款”后，crontab更新status为”过期未支付”，new-php读取到”过期未支付”，old-php乐观锁更新”已支付”没有生效抛出异常，new-php返回”拒绝付款”。

如果并发的逻辑更加复杂多样，该怎么应对呢？其实是有秘诀的，一方面是方法论正确，即提前梳理好所有状态的变迁图，每一个逻辑只管好自己的事情，遵循乐观锁思路进行编码，如果发生更新无效(或者mysql异常)，通过重试(例如crontab定期扫描mysql发现异常订单)来重新执行逻辑，直到成功执行为止。另一方面，在方法论正确的场景下，多枚举不同时序的并发场景，对不自信的环节动脑琢磨，或者通过构造异常代码进行验证的方法，最终保证代码正确性。

## 其他

乐观锁机制采取了更加宽松的加锁机制。悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。相对悲观锁而言，**乐观锁更倾向于开发运用**。

