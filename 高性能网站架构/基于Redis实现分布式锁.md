### 基于Redis实现分布式锁

最常见的一种方案就是使用Redis做分布式锁

使用Redis做分布式锁的思路大概是这样的：在redis中设置一个值表示加了锁，然后释放锁的时候就把这个key删除。

1. **根据lockKey区进行setnx（set not exist，如果key值为空，则正常设置，返回1，否则不会进行设置并返回0）操作，如果设置成功，表示已经获得锁，否则并没有获取锁。**
2. **如果没有获得锁，去Redis上拿到该key对应的值，在该key上我们存储一个时间戳（用毫秒表示，t1），为了避免死锁以及其他客户端占用该锁超过一定时间（5秒），使用该客户端当前时间戳，与存储的时间戳作比较。**
3. **如果没有超过该key的使用时限，返回false，表示其他人正在占用该key，不能强制使用；如果已经超过时限，那我们就可以进行解锁，使用我们的时间戳来代替该字段的值。**
4. **但是如果在setnx失败后，get该值却无法拿到该字段时，说明操作之前该锁已经被释放，这个时候，最好的办法就是重新执行一遍setnx方法来获取其值以获得该锁。**

　　**释放锁：删除redis中key**

具体代码是这样的：

```
// 获取锁
// NX是指如果key不存在就成功，key存在返回false，PX可以指定过期时间
SET anyLock unique_value NX PX 30000


// 释放锁：通过执行一段lua脚本
// 释放锁涉及到两条指令，这两条指令不是原子性的
// 需要用到redis的lua脚本支持特性，redis执行lua脚本是原子性的
if redis.call("get",KEYS[1]) == ARGV[1] then
   return redis.call("del",KEYS[1])
else
   return 0
end复制代码
```



这种方式有几大要点：

- 一定要用SET key value NX PX milliseconds 命令

  如果不用，先设置了值，再设置过期时间，这个不是原子性操作，有可能在设置过期时间之前宕机，会造成死锁(key永久存在)

- value要具有唯一性

  这个是为了在解锁的时候，需要验证value是和加锁的一致才删除key。

  这是避免了一种情况：假设A获取了锁，过期时间30s，此时35s之后，锁已经自动释放了，A去释放锁，但是此时可能B获取了锁。A客户端就不能删除B的锁了。

![img](http://qa0a9jn2t.bkt.clouddn.com/16bdef11fa0b11b0.jpeg)



除了要考虑客户端要怎么实现分布式锁之外，还需要考虑redis的部署问题。

redis有3种部署方式：

- 单机模式
- master-slave + sentinel选举模式
- redis cluster模式



使用redis做分布式锁的缺点在于：如果采用单机部署模式，会存在单点问题，只要redis故障了。加锁就不行了。

采用master-slave模式，加锁的时候只对一个节点加锁，即便通过sentinel做了高可用，但是如果master节点故障了，发生主从切换，此时就会有可能出现锁丢失的问题。

基于以上的考虑，其实redis的作者也考虑到这个问题，他提出了一个RedLock的算法，这个算法的意思大概是这样的：

假设redis的部署模式是redis cluster，总共有5个master节点，通过以下步骤获取一把锁：

- 获取当前时间戳，单位是毫秒
- 轮流尝试在每个master节点上创建锁，过期时间设置较短，一般就几十毫秒
- 尝试在大多数节点上建立一个锁，比如5个节点就要求是3个节点（n / 2 +1）
- 客户端计算建立好锁的时间，如果建立锁的时间小于超时时间，就算建立成功了
- 要是锁建立失败了，那么就依次删除这个锁
- 只要别人建立了一把分布式锁，你就得不断轮询去尝试获取锁



但是这样的这种算法还是颇具争议的，可能还会存在不少的问题，无法保证加锁的过程一定正确。


![img](http://qa0a9jn2t.bkt.clouddn.com/16bdef134fefb835.jpeg)

### 另一种方式：Redisson

此外，实现Redis的分布式锁，除了自己基于redis client原生api来实现之外，还可以使用开源框架：Redission

Redisson是一个企业级的开源Redis Client，也提供了分布式锁的支持。我也非常推荐大家使用，为什么呢？

回想一下上面说的，如果自己写代码来通过redis设置一个值，是通过下面这个命令设置的。

- SET anyLock unique_value NX PX 30000

这里设置的超时时间是30s，假如我超过30s都还没有完成业务逻辑的情况下，key会过期，其他线程有可能会获取到锁。

这样一来的话，第一个线程还没执行完业务逻辑，第二个线程进来了也会出现线程安全问题。所以我们还需要额外的去维护这个过期时间，太麻烦了~

我们来看看redisson是怎么实现的？先感受一下使用redission的爽：







```
Config config = new Config();
config.useClusterServers()
    .addNodeAddress("redis://192.168.31.101:7001")
    .addNodeAddress("redis://192.168.31.101:7002")
    .addNodeAddress("redis://192.168.31.101:7003")
    .addNodeAddress("redis://192.168.31.102:7001")
    .addNodeAddress("redis://192.168.31.102:7002")
    .addNodeAddress("redis://192.168.31.102:7003");

RedissonClient redisson = Redisson.create(config);


RLock lock = redisson.getLock("anyLock");
lock.lock();
lock.unlock();复制代码
```



就是这么简单，我们只需要通过它的api中的lock和unlock即可完成分布式锁，他帮我们考虑了很多细节：

- redisson所有指令都通过lua脚本执行，redis支持lua脚本原子性执行

- redisson设置一个key的默认过期时间为30s,如果某个客户端持有一个锁超过了30s怎么办？

  redisson中有一个`watchdog`的概念，翻译过来就是看门狗，它会在你获取锁之后，每隔10秒帮你把key的超时时间设为30s

  这样的话，就算一直持有锁也不会出现key过期了，其他线程获取到锁的问题了。

- redisson的“看门狗”逻辑保证了没有死锁发生。

  (如果机器宕机了，看门狗也就没了。此时就不会延长key的过期时间，到了30s之后就会自动过期了，其他线程可以获取到锁)

![img](http://qa0a9jn2t.bkt.clouddn.com/16bdef157e04045c.jpeg)

这里稍微贴出来其实现代码：







```
// 加锁逻辑
private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, final long threadId) {
    if (leaseTime != -1) {
        return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    }
    // 调用一段lua脚本，设置一些key、过期时间
    RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    ttlRemainingFuture.addListener(new FutureListener<Long>() {
        @Override
        public void operationComplete(Future<Long> future) throws Exception {
            if (!future.isSuccess()) {
                return;
            }
            
            Long ttlRemaining = future.getNow();
            // lock acquired
            if (ttlRemaining == null) {
                // 看门狗逻辑
                scheduleExpirationRenewal(threadId);
            }
        }
    });
    return ttlRemainingFuture;
}


<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);

    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
              "if (redis.call('exists', KEYS[1]) == 0) then " +
                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                  "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              "return redis.call('pttl', KEYS[1]);",
                Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}



// 看门狗最终会调用了这里
private void scheduleExpirationRenewal(final long threadId) {
    if (expirationRenewalMap.containsKey(getEntryName())) {
        return;
    }

    // 这个任务会延迟10s执行
    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            
            // 这个操作会将key的过期时间重新设置为30s
            RFuture<Boolean> future = renewExpirationAsync(threadId);
            
            future.addListener(new FutureListener<Boolean>() {
                @Override
                public void operationComplete(Future<Boolean> future) throws Exception {
                    expirationRenewalMap.remove(getEntryName());
                    if (!future.isSuccess()) {
                        log.error("Can't update lock " + getName() + " expiration", future.cause());
                        return;
                    }
                    
                    if (future.getNow()) {
                        // reschedule itself
                        // 通过递归调用本方法，无限循环延长过期时间
                        scheduleExpirationRenewal(threadId);
                    }
                }
            });
        }

    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);

    if (expirationRenewalMap.putIfAbsent(getEntryName(), new ExpirationEntry(threadId, task)) != null) {
        task.cancel();
    }
}复制代码
```

另外，redisson还提供了对redlock算法的支持,它的用法也很简单：

```
RedissonClient redisson = Redisson.create(config);
RLock lock1 = redisson.getFairLock("lock1");
RLock lock2 = redisson.getFairLock("lock2");
RLock lock3 = redisson.getFairLock("lock3");
RedissonRedLock multiLock = new RedissonRedLock(lock1, lock2, lock3);
multiLock.lock();
multiLock.unlock();
```










































