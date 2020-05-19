我们知道 Redis 的数据全部都存储在内存中，如果 Redis 服务器突然宕机，则会导致数据全部丢失，所以必须要有一种机制来保证 Redis 的数据不丢失或者丢失很少一部分，这种机制就是 Redis 的持久化机制。

**Redis 的持久化则是将内存中的数据持久备份到硬盘中，在服务重启时可以恢复**。Redis 目前提供了两种持久化方式：

> 1. RDB，即 Redis DataBase：把 Redis 服务器中内存的数据保存到一个 dump 文件中，数据的集合
> 2. AOF，即 Append-only file：把所有对 Redis 服务器进行修改的命令保存到一个 aof 文件中，命令的集合

下面就这两种方式来做详细说明。

## RDB

RDB，即 Redis 的内存快照，它是在某一个时间点将 Redis 的内存数据**全量**写入一个临时文件，当写入完成后，用该临时文件替换上一次持久化生成的文件，这样就完成了一次持久化过程。

RDB 相关配置

```
#dbfilename：持久化数据存储在本地的文件
dbfilename dump.rdb

#dir：持久化数据存储在本地的路径，如果是在/redis/src下启动的redis-cli，则数据会存储在当前src目录下
dir ./

##snapshot触发的时机，save <seconds> <changes>  
##一般来说我们需要根据系统变更操作密集程度来认真评估这个值
##可以通过 “save “””来关闭snapshot功能  
save 900 1
save 300 10
save 60 10000

##当snapshot时出现错误无法继续时，是否阻塞客户端“变更操作”，“错误”可能因为磁盘已满/磁盘故障/OS级别异常等 
stop-writes-on-bgsave-error yes

##是否启用rdb文件压缩，默认为“yes”，压缩往往意味着“额外的cpu消耗”，同时也意味这较小的文件尺寸以及较短的网络传输时间  
rdbcompression yes  
```

### 触发过程

RDB 持久化触发分为手动和自动两种方式，其中手动方式有两种：save 命令和 bgsave 命令。

- save 命令：阻塞 Redis 服务，直到整个 RDB 持久化完成。我们知道 RDB 是全量持久化，如果内存的数据量大，则造成长时间的阻塞，这样势必会影响业务。所以一般不推荐采用这种方式。
- bgsave 命令：该模式下的 RDB 持久化由子进程完成.Redis 进程接收到该命令后，会 fork 操作创建一个子进程,持久化过程有子进程完成。Redis 服务阻塞只会发生在 fork 阶段，而且该阶段时间过程一般都会很短。其流程如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301001.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903301001.png)

1. 客户端发送 bgsave 命令，Redis 进程首先判断当前是否存在其他子进程在执行操作，如 RDB 或者 AOF 子进程，如果有，则立刻返回，否则执行 2。
2. Redis 父进程执行 fork 操作创建子进程，在 fork 操作过程中父进程会阻塞。
3. Redis 父进程 fork 操作完成后，bgsave 命令返回 `Background saving started` 信息并不再阻塞 Redis 父进程，可以继续响应其他命令了。
4. fork 的子进程则根据 Redis 父进程的内存数据生成 RDB 文件，完成后替换原有的 RDB 文件。同时，发送信号给 Redis 父进程表示 RDB 操作已完成，父进程则更新统计信息。

下面我们简单演示下，bgsave 命令过程

我们先看系统中原有的 rdb 文件（在 src 目录下，文件名为 dump.rdb），如下图：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301001.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903301002.png)

先 `set todo test_bgsave`，然后执行 bgsave 命令，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301001.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903301003.png)

再次查看 dump.rdb 文件内容，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903301005.png)

大家会发现该文件内容比第一次看的文件内容多了一个 “todo test_bgsave”，这就证明 bgsave 生成 RDB 文件成功。

在生成 RDB 文件过程中，会打印相关的 Redis 日志，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903301004.png)

我们不能总是通过 bgsave 命令来手动触发 RDB 持久化吧，否则这得多累人啊，而且也是不可能完成的事情啊，所以就有了自动触发的方式：`save m n`

**save m n**

该方式在 redis.conf 中进行了说明，m 表示“间隔时间”，n 表示 “变更次数”，只有同时符合这两个条件才会触发，否则“变更次数”会被继续累加到下一个“间隔时间”上。同时，该方式也不会阻塞。这个可以认为是 bgsave 的自动触发过程。

```
save 900 1
save 300 10
save 60 10000
```

这个表示更改了 1 个 key 时间隔 900s 进行持久化存储，更改了 10 个 key 时间间隔 300s 进行持久化存储，更改了 10000 个 key 时间间隔 60s 进行持久化存储。触发写入的 Redis 的日志如下

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201903251002.png)

### RDB 数据恢复

RDB 文件的载入工作是在服务器启动时自动加载的，如果在 Redis 服务器中没有设置 AOF ，那么 Redis 服务器在启动是就会检测 RDB 文件（redis-check-rdb 命令），并自动载入。在载入期间，Redis 服务器会一直处于阻塞状态，直到完成为止。如果载入的 RDB 文件损坏了，则会载入失败，Redis 服务会启动失败，我们可以通过 `redis-check-rdb` 来完成对 RDB 文件的检测和修复。

### 优缺点

- 优点
  - 由于 RDB 文件是一个非常紧凑的二进制文件，所以加载的速度回快于 AOF 方式
  - fork 子进程方式，不会阻塞
  - RDB 文件代表着 Redis 服务器的某一个时刻的全量数据，所以它非常适合做冷备份和全量复制的场景
- 缺点
  - 没办法做到实时持久化，会存在丢数据的风险。定时执行持久化过程，如果在这个过程中服务器崩溃了，则会导致这段时间的数据全部丢失

## AOF

RDB 最大的问题就在于他提供的持久化策略是不安全的，不适合做实时持久化，所以 Redis 提供了第二套持久化方式：AOF 来解决这个问题。

AOF 即 append only file，它是将每一行对 Redis 数据进行修改的命令以独立日志的方式存储起来。由于 Redis 是将“操作 + 数据” 以格式化的方式保存在日志文件中，他代表了这段时间所有对 Redis 数据的的操作过程，所以在数据恢复时，我们可以直接 replay 该日志文件，即可还原所有操作过程，达到恢复数据的目的。它的主要目的是解决了数据持久化的实时性。

AOF 默认关闭，需要在配置文件 redis.conf 中开启，`appendonly yes`。与 AOF 相关的配置如下：

```
## aof功能的开关，默认为“no”，修改为 “yes” 开启  
## 只有在“yes”下，aof重写/文件同步等特性才会生效  
appendonly no  

## 指定aof文件名称  
appendfilename appendonly.aof  

## 指定aof操作中文件同步策略，有三个合法值：always everysec no,默认为everysec  
# appendfsync always  
 appendfsync everysec 
# appendfsync no 

##在aof-rewrite期间，appendfsync是否暂缓文件同步，"no"表示“不暂缓”，“yes”表示“暂缓”，默认为“no”  
no-appendfsync-on-rewrite no  

## aof文件rewrite触发的最小文件尺寸(mb,gb),只有大于此aof文件大于此尺寸是才会触发rewrite，默认“64mb”，建议“512mb”  
auto-aof-rewrite-min-size 64mb  

## 相对于“上一次”rewrite，本次rewrite触发时aof文件应该增长的百分比。  
## 每一次rewrite之后，redis都会记录下此时“新aof”文件的大小(例如A)，那么当aof文件增长到A*(1 + p)之后  
## 触发下一次rewrite，每一次aof记录的添加，都会检测当前aof文件的尺寸。  
auto-aof-rewrite-percentage 100  
```

### AOF 执行流程

AOF 总共分为三个流程

1. 命令写入
2. 文件同步
3. 文件重写

**命令写入**

Redis 在命令写入时，将缓冲区（aof_buf）引用进来了。我们知道 Redis 是单线程的，如果每次 append aof 文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘的负载，性能肯定会受到一些影响，所以将命令先写入到 aof_buf 中。这样做还有一个目的，那就 Redis 可以提供多种同步策略，让用户在性能和安全方面做出平衡。

**文件同步**

命令写入到缓冲区，然后根据不同的策略刷到硬盘中。Redis 提供提供了三种不同的同步策略：always everysec no，由 appendfsync 控制。

| 策略 |             always             |        everysec         |                   no                   |
| :--: | :----------------------------: | :---------------------: | :------------------------------------: |
| 行为 |      每条命令fsync到硬盘       | 每秒把缓冲区fsybc到硬盘 | OS决定什么时候来把缓冲区的命令写到硬盘 |
| 特点 | 不丢失数据，IO开销大，硬盘压力 | 可能会丢失某一秒的数据  |             不用管，不可控             |

> - always :每天命令都会同步至硬盘，是最安全的方式，但是对 IO 开支大，硬盘压力大，无法满足 Redis 高性能的要求，所以我们一般不推荐这种策略。如果对数据安全性要求这么高，其实可以选择关系型数据库。
> - everyesc：每秒同步一次，算是一种比较中庸的选择方式，也是 Redis 推荐的方式，但是如果遇到服务器故障，可能会丢失最近一秒的记录 。
> - no：Redis 服务并不会直接参与同步，而是将控制权交个操作系统，操作系统会根据系统实际情况来触发同步，不可控。

**文件重写**

随着命令的不断写入，AOF 文件会越来越庞大，直接的影响就是导致“数据恢复”时间延长，而且有些历史的操作是可以废弃的（比如超时、del等等），为了解决这些问题，Redis 提供了 “文件重写”功能，该功能有以下两种方式触发。

1. 手动触发：bgrewriteaof 命令
2. 自动触发：由 auto-aof-rewrite-min-size 和 auto-aof-rewrite-percentage 参数来确定自动触发时机。
   - auto-aof-rewrite-min-size：运行 AOF 重写时文件最小体积
   - auto-aof-rewrite-percentage：代表当前 AOF 文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值
   - 触发时机 = 当前 AOF 文件空间 > AOF 重写时文件最小体积 &&

重写 AOF 文件最直观的表现是导致 AOF 文件减小，重写时，Redis 主要做了如下几件事情让 AOF 文件减小：

1. 已过期的数据不在写入文件。
2. 保留最终命令。例如 `set key1 value1 `、`set key1 value2`、....`set key1 valuen`，类似于这样的命令，只需要保留最后一个即可。
3. 删除无用的命令。例如 `set key1 valuel;del key1`,这样的命令也是可以不用写入文件中的。
4. 多条命令合并成一条命令。例如 `lpush list a、lpush list b、lpush list c`，可以转化为 `lpush list a b c`

文件重写流程如下（参考 《Redis 开发与运维》）：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903301002.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904031001.png)

1. Redis 服务接收到 bgrewriteaof 命令的时候，会做两步检查。
   - 如果当前进程正在执行 AOF 重写，则直接返回。
   - 如果有进程正在执行 bgsave，那么需要等待 bgsave 执行完毕后再执行 AOF 重写。
2. Redis 进程会 fork 一个子进程执行 AOF 重写，成功后，Redis 服务继续响应命令，不会影响 Redis 原有的 AOF 流程（即命令写入 aof_buf 缓冲区和缓冲区的数据刷进硬盘）。
3. 在子进程重写过程中，Redis 主进程会将受到的命令也会写入 AOF 重写缓冲区，这个缓冲区和 aof_buf 缓冲区不一样，需要区分，这样做的目的是为了防止重写过程中数据的丢失。
4. 由于使用写时复制技术（copy-on-write），子进程只能拿到父进程在 fork 子进程时刻的文件进行重写。子进程根据内存快照，按照命令重写规则将命令重写到新的文件中。这里需要注意的是每次写入硬盘的数据量不能太大，否则容易导致硬盘阻塞，该值由 aof-rewrite-incremental-fsync 控制，默认为 32M。
5. 子进程完成 AOF 重写后会给发消息给 Redis 主进程，主进程则会将 AOF 重写缓冲区的数据写进新的文件，然后用新的 AOF 文件 替换老的文件。
6. 完成 AOF 重写。

在整个重写过程中，我觉得会有三个地方会阻塞 Redis：

1. fork 子进程阶段，开销与 bgsave 一致。
2. 主进程将重写缓冲区的数据写入到新的 AOF 文件。
3. 用新的 AOF 文件替换来的 AOF 文件。

### AOF 演示

现在 AOF 的原理基本已经掌握差不多了，下面小编将整个 AOF 过程演示一遍。

1. 开启 AOF 持久化

因为 Redis 默认是关闭 AOF 的，所以我们需要手动开启 AOF ，如下：

```
appendonly yes
```

2、输入命令，等待生成 AOF 文件

同步策略我们选择的是 everysec ，即一秒钟就执行一次同步，为了演示效果，我们通过程序来完成命令的写入，如下：

```java
    public static void main(String[] args){
        Jedis jedis = new Jedis("localhost");

        jedis.set("name","chenssy1");
        jedis.set("birthday","02-31");

        jedis.set("name","chenssy2");
        jedis.set("name","chenssy3");

        jedis.del("birthday");

        jedis.lpush("listkey","value1");
        jedis.lpush("listkey","value2");
        jedis.lpush("listkey","value3");
    }
```

查看 AOF 文件内容，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201903251002.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904031002.jpg)

AOF 文件中的内容与输入的命令一一对应。

3、重写 AOF 文件

按照上面的重写规则，我们知道重写完成后，AOF 里面应该只有两条命令，如下：

```
set name chenssy3
lpush listkey value1 value2 value3
```

执行 bgrewriteaof 命令后，Redis 会打印如下日志：

```
1835:M 03 Apr 2019 22:02:50.092 * Background append only file rewriting started by pid 1965
1835:M 03 Apr 2019 22:02:50.116 * AOF rewrite child asks to stop sending diffs.
1965:C 03 Apr 2019 22:02:50.116 * Parent agreed to stop sending diffs. Finalizing AOF...
1965:C 03 Apr 2019 22:02:50.117 * Concatenating 0.00 MB of AOF diff received from parent.
1965:C 03 Apr 2019 22:02:50.117 * SYNC append only file rewrite performed
1835:M 03 Apr 2019 22:02:50.191 * Background AOF rewrite terminated with success
1835:M 03 Apr 2019 22:02:50.191 * Residual parent diff successfully flushed to the rewritten AOF (0.00 MB)
1835:M 03 Apr 2019 22:02:50.192 * Background AOF rewrite finished successfully
```

再次查看 aof 文件，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201904031001.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904031003.png)

由于是二进制文件，所以就没有刚刚那么直观了，但是隐约可以看到结果和我们预测的一致。

### AOF 的优缺点

**优点**

- 相比于 RDB，AOF 更加安全，默认同步策略为 everysec 即每秒同步一次，所以顶多我们就失去一秒的数据（其实不止一秒，这个话题我们后面分析）。
- 根据关注点不同，AOF 提供了不同的同步策略，我们可以根据自己的需求来选择。
- AOF 文件是以 append-only 方式写入，相比如 RDB 全量写入的方式，它没有任何磁盘寻址的开销，写入性能非常高。

**缺点**

- 由于 AOF 日志文件是命令级别的，所以相比于 RDB 紧致的二进制文件而言它的加载速度会慢些。
- AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低。

### AOF 追加阻塞

AOF 默认的持久化策略是 everysec，它是每隔一秒钟执行一次 AOF。对于这种定时处理的方式，Redis 是使用另外一个线程来处理。在系统硬盘资源紧张的时候，会造成 Redis 主线程阻塞，如下（图参考 《Redis 开发与运维》）：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201904031003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904061001.png)

从上图我们可以得到如下几个结论：

1. everysec 持久化策略最多丢失的不是 1 秒的数据，而是 2 秒
2. Redis 使用另外一个线程来进行缓存持久化操作，同时会记录该时间
3. 如果系统持久化缓慢，将会导致 Redis 主线程阻塞影响效率

## RDB 和 AOF 混合模式

通过上面的介绍我们知道了 RDB 和 AOF 各有自己的优缺点，选择任意其一都需要接受他的缺点：

- RDB 能够快速地存储和恢复数据，但是在服务器宕机时会丢失大量的数据，没有保证数据的实时性和安全性
- AOF 能够实时持久化数据并且提高了数据的安全性，但是在存储和恢复数据方面又会消耗大量时间

那有没有“鱼和熊掌可兼得”的方案呢？有！！ Redis 4.0 推出了 **RDB-AOF 混合持久化**方案，该方案是在 AOF 重写阶段创建一个同时包含 RDB 数据和 AOF 数据的 AOF 文件，其中 RDB 数据位于 AOF 文件的开头，他存储了服务器开始执行重写操作时 Redis 服务器的数据状态（RDB 快照方案），重写操作执行之后的 Redis 命令，则会继续 append 在 AOF 文件末尾，即 RDB 数据之后（AOF 日志追加方案），一般这部分数据都会比较小。这样在 Redis 重启的时候，则可以先加载 RDB 的内容，然后再加载 AOF 的日志内容，这样重启的效率则会得到很大的提升，而且由于在运行阶段 Redis 命令都会以 append 的方式写入 AOF 文件，保证了数据的实时性和安全性。这就是“鱼和熊掌均可的”的方案。

下面我们简单模拟下。要开启混合持久化需要同时将 `appendonly` 和 `aof-use-rdb-preamble` 都设置为 yes 。程序同样利用 AOF 的程序演示：

1、执行程序，输入数据

2、 执行 bgrewriteaof 命令，开启重写，查看 appendonly.aof 内容，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201904061003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904061003.png)

内容和在演示 RDB 是的内容一致，从内容中我们可以隐约看到 name 和 listkey 两个 key 值

3、输入如下两个命令

```
set birthday 02-31
set location shenzhen
```

然后再看其中的内容，如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201904061003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/sk-redis/redis-201904061004.png)

各位小伙伴还可以继续往下走，执行 bgrewriteaof 命令，看是不是已经为 RDB 内容格式，这里小编就不过多演示了。

所以，通过 RDB-AOF 混合持久化方案，我们可以得到 RDB 加载数据快和 AOF 保证数据安全性和实时性的两个持久化方案的优点。

## Redis 加载

Redis 重启加载数据的流程图如下（参考《Redis 开发与运维》）：

注：此图有错误，需要修改

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/img/redis-201904061004.png)](http://cmsblogs.com/wp-content/resources/image.cmsblogs/sike-java/sike-redis/redis-202003311001.png)