- [基于zookeeper实现分布式锁](#--zookeeper------)
  * [ZK的四种节点](#zk-----)
  * [排它锁的实现](#------)
    + [代码实现](#----)
    + [可重入性排他锁如何设计](#-----------)
  * [读写锁的实现](#------)
    + [读锁的设计](#-----)
    + [写锁的设计](#-----)
    + [如何监听](#----)
    + [代码实现](#-----1)
  * [Curator实现](#curator--)

# 基于zookeeper实现分布式锁

常见的分布式锁实现方案里面，除了使用redis来实现之外，使用zookeeper也可以实现分布式锁。

在介绍zookeeper(下文用zk代替)实现分布式锁的机制之前，先粗略介绍一下zk是什么东西：

Zookeeper是一种提供配置管理、分布式协同以及命名的中心化服务。

zk的模型是这样的：zk包含一系列的节点，叫做znode，就好像文件系统一样每个znode表示一个目录，然后znode有一些特性：

- 有序节点：假如当前有一个父节点为`/lock`，我们可以在这个父节点下面创建子节点；

  zookeeper提供了一个可选的有序特性，例如我们可以创建子节点“/lock/node-”并且指明有序，那么zookeeper在生成子节点时会根据当前的子节点数量自动添加整数序号

  也就是说，如果是第一个创建的子节点，那么生成的子节点为`/lock/node-0000000000`，下一个节点则为`/lock/node-0000000001`，依次类推。

  

- 临时节点：客户端可以建立一个临时节点，在会话结束或者会话超时后，zookeeper会自动删除该节点。

  

- 事件监听：在读取数据时，我们可以同时对节点设置事件监听，当节点数据或结构变化时，zookeeper会通知客户端。当前zookeeper有如下四种事件：

- 节点创建
- 节点删除
- 节点数据修改
- 子节点变更



基于以上的一些zk的特性，我们很容易得出使用zk实现分布式锁的落地方案：



1. **使用zk的临时节点和有序节点，每个线程获取锁就是在zk创建一个临时有序的节点，比如在/lock/目录下。**

2. **创建节点成功后，获取/lock目录下的所有临时节点，再判断当前线程创建的节点是否是所有的节点的序号最小的节点**

3. **如果当前线程创建的节点是所有节点序号最小的节点，则认为获取锁成功。**

4. **如果当前线程创建的节点不是所有节点序号最小的节点，则对节点序号的前一个节点添加一个事件监听。**

   **比如当前线程获取到的节点序号为`/lock/003`,然后所有的节点列表为`[/lock/001,/lock/002,/lock/003]`,则对`/lock/002`这个节点添加一个事件监听器。**



如果锁释放了，会唤醒下一个序号的节点，然后重新执行第3步，判断是否自己的节点序号是最小。

比如`/lock/001`释放了，`/lock/002`监听到时间，此时节点集合为`[/lock/002,/lock/003]`,则`/lock/002`为最小序号节点，获取到锁。



整个过程如下：


![img](https://gitee.com/cdx_dayshow/picBed/raw/master/16bdef1768e6e969.jpeg)

代码思路

## ZK的四种节点

- 持久性节点：节点创建后将会一直存在
- 临时节点：临时节点的生命周期和当前会话绑定，一旦当前会话断开临时节点也会删除，当然可以主动删除。
- 持久有序节点：节点创建一直存在，并且zk会自动为节点加上一个自增的后缀作为新的节点名称。
- 临时有序节点：保留临时节点的特性，并且zk会自动为节点加上一个自增的后缀作为新的节点名称。



## 排它锁的实现

- 排他锁的实现相对简单一点，利用了**zk的创建节点不能重名的特性**。如下图：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/zk排他锁.png)

- 根据上图分析大致分为如下步骤：
  1. 尝试获取锁：创建`临时节点`，zk会保证只有一个客户端创建成功。
  2. 创建临时节点成功，获取锁成功，执行业务逻辑，业务执行完成后删除锁。
  3. 创建临时节点失败，阻塞等待。
  4. 监听删除事件，一旦临时节点删除了，表示互斥操作完成了，可以再次尝试获取锁。
  5. 递归：获取锁的过程是一个递归的操作，`获取锁->监听->获取锁`。
- **如何避免死锁**：创建的是临时节点，当服务宕机会话关闭后临时节点将会被删除，锁自动释放。

### 代码实现

- 作者参照JDK锁的实现方式加上模板方法模式的封装，封装接口如下：

```
/**
 * @Description ZK分布式锁的接口
 * @Author 陈某
 * @Date 2020/4/7 22:52
 */
public interface ZKLock {
    /**
     * 获取锁
     */
    void lock() throws Exception;

    /**
     * 解锁
     */
    void unlock() throws Exception;
}
```

- 模板抽象类如下：

```
/**
 * @Description 排他锁，模板类
 * @Author 陈某
 * @Date 2020/4/7 22:55
 */
public abstract class AbstractZKLockMutex implements ZKLock {

    /**
     * 节点路径
     */
    protected String lockPath;

    /**
     * zk客户端
     */
    protected CuratorFramework zkClient;

    private AbstractZKLockMutex(){}

    public AbstractZKLockMutex(String lockPath,CuratorFramework client){
        this.lockPath=lockPath;
        this.zkClient=client;
    }

    /**
     * 模板方法，搭建的获取锁的框架，具体逻辑交于子类实现
     * @throws Exception
     */
    @Override
    public final void lock() throws Exception {
        //获取锁成功
        if (tryLock()){
            System.out.println(Thread.currentThread().getName()+"获取锁成功");
        }else{  //获取锁失败
            //阻塞一直等待
            waitLock();
            //递归，再次获取锁
            lock();
        }
    }

    /**
     * 尝试获取锁，子类实现
     */
    protected abstract boolean tryLock() ;


    /**
     * 等待获取锁，子类实现
     */
    protected abstract void waitLock() throws Exception;


    /**
     * 解锁：删除节点或者直接断开连接
     */
    @Override
    public  abstract void unlock() throws Exception;
}
```

- 排他锁的具体实现类如下：

```
/**
 * @Description 排他锁的实现类，继承模板类 AbstractZKLockMutex
 * @Author 陈某
 * @Date 2020/4/7 23:23
 */
@Data
public class ZKLockMutex extends AbstractZKLockMutex {

    /**
     * 用于实现线程阻塞
     */
    private CountDownLatch countDownLatch;

    public ZKLockMutex(String lockPath,CuratorFramework zkClient){
        super(lockPath,zkClient);
    }

    /**
     * 尝试获取锁：直接创建一个临时节点，如果这个节点存在创建失败抛出异常，表示已经互斥了，
     * 反之创建成功
     * @throws Exception
     */
    @Override
    protected boolean tryLock()  {
        try {
            zkClient.create()
                    //临时节点
                    .withMode(CreateMode.EPHEMERAL)
                    //权限列表 world:anyone:crdwa
                    .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                    .forPath(lockPath,"lock".getBytes());
            return true;
        }catch (Exception ex){
            return false;
        }
    }


    /**
     * 等待锁，一直阻塞监听
     * @return  成功获取锁返回true，反之返回false
     */
    @Override
    protected void waitLock() throws Exception {
        //监听节点的新增、更新、删除
        final NodeCache nodeCache = new NodeCache(zkClient, lockPath);
        //启动监听
        nodeCache.start();
        ListenerContainer<NodeCacheListener> listenable = nodeCache.getListenable();

        //监听器
        NodeCacheListener listener=()-> {
            //节点被删除，此时获取锁
            if (nodeCache.getCurrentData() == null) {
                //countDownLatch不为null，表示节点存在，此时监听到节点删除了，因此-1
                if (countDownLatch != null)
                    countDownLatch.countDown();
            }
        };
        //添加监听器
        listenable.addListener(listener);

        //判断节点是否存在
        Stat stat = zkClient.checkExists().forPath(lockPath);
        //节点存在
        if (stat!=null){
            countDownLatch=new CountDownLatch(1);
            //阻塞主线程，监听
            countDownLatch.await();
        }
        //移除监听器
        listenable.removeListener(listener);
    }

    /**
     * 解锁，直接删除节点
     * @throws Exception
     */
    @Override
    public void unlock() throws Exception {
        zkClient.delete().forPath(lockPath);
    }
}
```

### 可重入性排他锁如何设计

- 可重入的逻辑很简单，在本地保存一个`ConcurrentMap`，`key`是当前线程，`value`是定义的数据，结构如下：

```
private final ConcurrentMap<Thread, LockData> threadData = Maps.newConcurrentMap();
```

- 重入的伪代码如下：

```
public boolean tryLock(){
    //判断当前线程是否在threadData保存过
    //存在，直接return true
    //不存在执行获取锁的逻辑
    //获取成功保存在threadData中
}
```

## 读写锁的实现

- 读写锁分为读锁和写锁，区别如下：
  - 读锁允许多个线程同时读数据，但是在读的同时不允许写线程修改。
  - 写锁在获取后，不允许多个线程同时写或者读。
- 如何实现读写锁？ZK中有一类节点叫临时有序节点，上文有介绍。下面我们来利用临时有序节点来实现读写锁的功能。

### 读锁的设计

- 读锁允许多个线程同时进行读，并且在读的同时不允许线程进行写操作，实现原理如下图：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/读锁.png)

- 根据上图，获取一个读锁分为以下步骤：
  1. 创建临时有序节点（当前线程拥有的`读锁`或称作`读节点`）。
  2. 获取路径下所有的子节点，并进行`从小到大`排序
  3. 获取当前节点前的临近写节点(写锁)。
  4. 如果不存在的临近写节点，则成功获取读锁。
  5. 如果存在临近写节点，对其监听删除事件。
  6. 一旦监听到删除事件，**重复2,3,4,5的步骤(递归)**。

### 写锁的设计

- 线程一旦获取了写锁，不允许其他线程读和写。实现原理如下：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/写锁.png)

- 从上图可以看出唯一和写锁不同的就是监听的节点，这里是监听临近节点(读节点或者写节点)，读锁只需要监听写节点，步骤如下：
  1. 创建临时有序节点（当前线程拥有的`写锁`或称作`写节点`）。
  2. 获取路径下的所有子节点，并进行`从小到大`排序。
  3. 获取当前节点的临近节点(读节点和写节点)。
  4. 如果不存在临近节点，则成功获取锁。
  5. 如果存在临近节点，对其进行监听删除事件。
  6. 一旦监听到删除事件，**重复2,3,4,5的步骤(递归)**。

### 如何监听

- 无论是写锁还是读锁都需要监听前面的节点，不同的是读锁只监听临近的写节点，写锁是监听临近的所有节点，抽象出来看其实是一种链式的监听，如下图：

[![img](https://gitee.com/cdx_dayshow/picBed/raw/master/链式.png)

- 每一个节点都在监听前面的临近节点，一旦前面一个节点删除了，再从新排序后监听前面的节点，这样递归下去。

### 代码实现

- 作者简单的写了读写锁的实现，先造出来再优化，不建议用在生产环境。代码如下：

```
public class ZKLockRW  {

    /**
     * 节点路径
     */
    protected String lockPath;

    /**
     * zk客户端
     */
    protected CuratorFramework zkClient;

    /**
     * 用于阻塞线程
     */
    private CountDownLatch countDownLatch=new CountDownLatch(1);


    private final static String WRITE_NAME="_W_LOCK";

    private final static String READ_NAME="_R_LOCK";


    public ZKLockRW(String lockPath, CuratorFramework client) {
        this.lockPath=lockPath;
        this.zkClient=client;
    }

    /**
     * 获取锁，如果获取失败一直阻塞
     * @throws Exception
     */
    public void lock() throws Exception {
        //创建节点
        String node = createNode();
        //阻塞等待获取锁
        tryLock(node);
        countDownLatch.await();
    }

    /**
     * 创建临时有序节点
     * @return
     * @throws Exception
     */
    private String createNode() throws Exception {
        //创建临时有序节点
       return zkClient.create()
                .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath(lockPath);
    }

    /**
     * 获取写锁
     * @return
     */
    public  ZKLockRW writeLock(){
        return new ZKLockRW(lockPath+WRITE_NAME,zkClient);
    }

    /**
     * 获取读锁
     * @return
     */
    public  ZKLockRW readLock(){
        return new ZKLockRW(lockPath+READ_NAME,zkClient);
    }

    private void tryLock(String nodePath) throws Exception {
        //获取所有的子节点
        List<String> childPaths = zkClient.getChildren()
                .forPath("/")
                .stream().sorted().map(o->"/"+o).collect(Collectors.toList());


        //第一个节点就是当前的锁，直接获取锁。递归结束的条件
        if (nodePath.equals(childPaths.get(0))){
            countDownLatch.countDown();
            return;
        }

        //1. 读锁：监听最前面的写锁，写锁释放了，自然能够读了
        if (nodePath.contains(READ_NAME)){
            //查找临近的写锁
            String preNode = getNearWriteNode(childPaths, childPaths.indexOf(nodePath));
            if (preNode==null){
                countDownLatch.countDown();
                return;
            }
            NodeCache nodeCache=new NodeCache(zkClient,preNode);
            nodeCache.start();
            ListenerContainer<NodeCacheListener> listenable = nodeCache.getListenable();
            listenable.addListener(() -> {
                //节点删除事件
                if (nodeCache.getCurrentData()==null){
                    //继续监听前一个节点
                    String nearWriteNode = getNearWriteNode(childPaths, childPaths.indexOf(preNode));
                    if (nearWriteNode==null){
                        countDownLatch.countDown();
                        return;
                    }
                    tryLock(nearWriteNode);
                }
            });
        }

        //如果是写锁，前面无论是什么锁都不能读，直接循环监听上一个节点即可，直到前面无锁
        if (nodePath.contains(WRITE_NAME)){
            String preNode = childPaths.get(childPaths.indexOf(nodePath) - 1);
            NodeCache nodeCache=new NodeCache(zkClient,preNode);
            nodeCache.start();
            ListenerContainer<NodeCacheListener> listenable = nodeCache.getListenable();
            listenable.addListener(() -> {
                //节点删除事件
                if (nodeCache.getCurrentData()==null){
                    //继续监听前一个节点
                    tryLock(childPaths.get(childPaths.indexOf(preNode) - 1<0?0:childPaths.indexOf(preNode) - 1));
                }
            });
        }
    }

    /**
     * 查找临近的写节点
     * @param childPath 全部的子节点
     * @param index 右边界
     * @return
     */
    private String  getNearWriteNode(List<String> childPath,Integer index){
        for (int i = 0; i < index; i++) {
            String node = childPath.get(i);
            if (node.contains(WRITE_NAME))
                return node;

        }
        return null;
    }

}
```





## Curator实现

Curator是一个zookeeper的开源客户端，也提供了分布式锁的实现。

- Curator是Netflix公司开源的一个Zookeeper客户端，与Zookeeper提供的原生客户端相比，Curator的抽象层次更高，简化了Zookeeper客户端的开发量。
- Curator在分布式锁方面已经为我们封装好了，大致实现的思路就是按照作者上述的思路实现的。中小型互联网公司还是建议直接使用框架封装好的，毕竟稳定，有些大型的互联公司都是手写的，牛逼啊。
- 创建一个排他锁很简单，如下：

- 创建一个排他锁很简单，如下：

```
//arg1：CuratorFramework连接对象，arg2：节点路径
lock=new InterProcessMutex(client,path);
//获取锁
lock.acquire();
//释放锁
lock.release();
```

其实现分布式锁的核心源码如下：

```
private boolean internalLockLoop(long startMillis, Long millisToWait, String ourPath) throws Exception
{
    boolean  haveTheLock = false;
    boolean  doDelete = false;
    try {
        if ( revocable.get() != null ) {
            client.getData().usingWatcher(revocableWatcher).forPath(ourPath);
        }

        while ( (client.getState() == CuratorFrameworkState.STARTED) && !haveTheLock ) {
            // 获取当前所有节点排序后的集合
            List<String>        children = getSortedChildren();
            // 获取当前节点的名称
            String              sequenceNodeName = ourPath.substring(basePath.length() + 1); // +1 to include the slash
            // 判断当前节点是否是最小的节点
            PredicateResults    predicateResults = driver.getsTheLock(client, children, sequenceNodeName, maxLeases);
            if ( predicateResults.getsTheLock() ) {
                // 获取到锁
                haveTheLock = true;
            } else {
                // 没获取到锁，对当前节点的上一个节点注册一个监听器
                String  previousSequencePath = basePath + "/" + predicateResults.getPathToWatch();
                synchronized(this){
                    Stat stat = client.checkExists().usingWatcher(watcher).forPath(previousSequencePath);
                    if ( stat != null ){
                        if ( millisToWait != null ){
                            millisToWait -= (System.currentTimeMillis() - startMillis);
                            startMillis = System.currentTimeMillis();
                            if ( millisToWait <= 0 ){
                                doDelete = true;    // timed out - delete our node
                                break;
                            }
                            wait(millisToWait);
                        }else{
                            wait();
                        }
                    }
                }
                // else it may have been deleted (i.e. lock released). Try to acquire again
            }
        }
    }
    catch ( Exception e ) {
        doDelete = true;
        throw e;
    } finally{
        if ( doDelete ){
            deleteOurPath(ourPath);
        }
    }
    return haveTheLock;
}复制代码
```



