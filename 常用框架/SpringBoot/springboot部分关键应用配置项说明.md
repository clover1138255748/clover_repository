因此根据官方文档的解释，将其翻译并加以理解，记录至此，供各位查看。（springboot版本基于 2.04）

## 一、Tomcat

```
server:
  tomcat:
  	...
复制代码
```

- accept-count ：当所有处理线程都被使用时，传入连接未处理请求的最大队列长度。 超过数量请求会被拒绝。
- basedir ： Tomcat基本目录。如果未指定，则使用一个临时目录
- connection-timeout ： 连接器在接受连接后将等待呈现请求URI行的时间 ，也就数说，本次连接多久没有数据传过来就会断开连接。
- max-connections ： 服务器在任何给定时间接受和处理的最大连接数。一旦达到限制，操作系统仍可以基于“ acceptCount”属性接受连接 ，默认为10000，一般最大连接数要大于最大线程数+最大等待数
- max-threads ： 工作线程的最大数量（默认为200），针对频繁IO的应用应该加大一点最大线程数，对于计算密集型的应用应该减小最大线程数。
- min-spare-threads ： 最小备用线程数，初始化时的备用线程数 （默认为10）
- max-http-form-post-size ：post请求表单最大大小，与 `spring.servlet.multipart.max-file-size` 一致，与 `spring.servlet.multipart.max-file-size` 及 `spring.servlet.multipart.max-request-size` 是没有关系的（默认为2MB）
- max-swallow-size ： 可吞下的请求正文的最大数量，与 `spring.servlet.multipart.max-request-size` 一致 ，与 `spring.servlet.multipart.max-file-size` 及 `spring.servlet.multipart.max-request-size` 是没有关系的（默认为2MB）

## 二、Undertow

```
server:
	undertow:
		...
复制代码
```

- always-set-keep-alive：设置时，是否将 `Connection: keep-alive` 头部添加到所有连接中。（2.04中不存在）
- buffer-size：IO 中所设置的 buffer 大小，单位为 byte。官方推荐值为 16k，也就是 `write()` 方法能够接受的最大值。默认值取决于 JVM 中的最大内存
- direct-buffers：是否在堆外分配buffer，默认值取决于 JVM 中的最大内存
- decode-url：url 是否需要进行 decode 处理（2.04中不存在）
- max-http-post-size：默认值为-1B，代表不设限制
- io-threads：分配给 worker 的 IO 线程数，默认值取决于最大的处理器数量（默认的配置上和机器中有效的处理器数相同，最小为2）
- worker-threads：worker 线程的数量，默认值是 io-threads 的八倍

官方上对 worker 的解释：

> WORKER_IO_THREADS
>
> The number of IO threads to create. IO threads perform non blocking tasks, and should never perform blocking operations because they are responsible for multiple connections, so while the operation is blocking other connections will essentially hang. **Two IO threads per CPU core is a reasonable default.**
>
> WORKER_TASK_CORE_THREADS
>
> The number of threads in the workers blocking task thread pool. When performing blocking operations such as Servlet requests threads from this pool will be used. In general it is hard to give a reasonable default for this, as it depends on the server workload. Generally this should be reasonably high, **around 10 per CPU core**.

## 三、Redis

```
spring:
  redis:
    ...
复制代码
```

- database ： db号 （默认为0）
- host ： Redis服务器主机 （默认为localhost）
- password ：  Redis服务器的登录密码
- port ：  Redis服务器端口
- timeout ： 连接超时
- shutdown-timeout ： 关机超时
- lettuce：
  - pool.max-active ：连接池中可以分配的最大连接数。使用负值表示没有限制。（默认为8）
  - pool.max-idle ：连接池最大的空闲连接数。使用负值表示无限数量的空闲连接。（默认为8）
  - pool.min-idle ：连接池最小的空闲连接数。仅当设置的此值和 `time between eviction runs` 为正值时，才生效（2.0 中不生效）
  - pool.time-between-eviction-runs： 该值表示，当为正值时，会开启一个监控线程，在 `time-between-eviction-runs` 时间中会执行一次空闲连接检查，并将空闲连接释放，直到连接数量为 `min-idle`（2.0 中不生效）
  - pool.max-wait ：连接池最大阻塞等待时间，指等待可用连接的最大等待时间，超过时间会抛出异常。 （默认为-1ms，指不限制）
  - pool.shutdown-timeout：客户端关闭超时时间，当客户端 client 在尝试关闭的时候，如果超过这个时间会抛出异常

## 四、MySQL

```
spring:
  datasource:
  	...
复制代码
```

- hikari：
  - auto-commit ：是否自动提交
  - connection-init-sql ：新连接初始化添加到连接池前时执行的语句，如果执行失败，默认会抛出异常且重试
  - connection-test-query ：这个设置是用作不支持 JDBC4 中 `Connection.isValid()` API 的驱动。当中的语句在连接从池中取出时，用于验证连接是否有效。**如果驱动程序支持JDBC4，建议不要设置此属性**（springboot2.04 中所使用为 JDBC4，所以不需要设置此值）
  - connection-timeout ：客户端请求（指应用）等待连接池连接的最大时间，如果超过此时间，会抛出 `SQLException` 异常。能接受的最小值为 250ms
  - driver-class-name ：在没有设置时，Hikari 会尝试根据 `jdbcUrl` 配置来决定所使用的驱动，但对于一些旧的驱动程序，必须指定driverClassName
  - idle-timeout ： 表示连接被设置为空闲连接的最大超时时间，这个值只有在 `minimumIdle` 小于 `maximumPoolSize` 时才会生效。一旦空闲连接数降为 `minimumIdle` 时，池中的空闲连接不会被释放。设置为 0 表示空闲连接永远不会从池中释放。 默认值为60000ms（10min）， 允许的最小值为 10000ms（10s）
  - max-lifetime ：连接在连接池中的最大存活时间。**建议设置此值，并且需要比所使用的数据库的连接时间限制少数秒**。0 值表示没有最大存活时间限制（永远存活）。最小允许为 30000ms（30s），默认值为 1800000ms（30min）
  - maximum-pool-size ：连接池的最大连接数，包括闲置和使用中的连接。当连接池到达此值并且没有多余的空闲连接时，调用获取连接方法 `getConnection()` 时，在超时前会阻塞至 `connectionTimeout` 时长，默认值为 10
  - minimum-idle ：连接池中维护的最小空闲连接数。推荐不设置此值，置连接池为 *fixed size* 连接池
  - read-only ：判断并设置连接池中的连接为只读模式
  - transaction-isolation ：设置连接池中连接的默认事务级别。如果没指定，所使用的默认事务级别与 JDBC 驱动定义的相同。所使用值在 `Connection` 中，例如 `TRANSACTION_READ_COMMITTED`、`TRANSACTION_REPEATABLE_READ`
  - validation-timeout ： 连接池中连接存活测试的最大时间，此超时时间必须小于 `connectionTimeout`，最小可接受值为 250ms，默认为 5000ms

## 后语

之后可能会对其他配置做一个详细的解释，如果上述解释有误，各位可以在下面留言反馈，我们会根据官方的说明再进行修正。


作者：广州芦苇科技Java开发团队
链接：https://juejin.im/post/5ef2da1df265da02ce218351
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。