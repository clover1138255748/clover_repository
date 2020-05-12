- [分布式系统定义](#-------)
- [分布式系统的难点](#--------)
- [ZooKeeper介绍](#zookeeper--)
- [安装ZooKeeper](#--zookeeper)
- [ZooKeeper集群配置](#zookeeper----)
- [最后](#--)

想必大家都对分布式系统有所耳闻，大部分人对分布式都能侃侃而谈，但到了真正实施的时候，才发现其中的不易。今天带大家一起了解一款开源软件，ZooKeeper。它通过一些简单好用的API，来解决分布式系统设计与开发中的难点。

ZooKeeper

这篇文章主要从下几个部分介绍：

- 什么是分布式系统和它的特性
- 分布式系统难在哪里
- ZooKeeper简介
- 下载安装ZooKeeper
- 客户端操作ZooKeeper
- ZooKeeper集群配置



### 分布式系统定义

> A distributed system is de ned as a software system that is composed of independent computing entities linked together by a computer network whose components communicate and coordinate with each other to achieve a common goal.
>  分布式系统是由独立的计算机通过网络连接在一起，并且通过一些组件来相互交流和协作来完成一个共同的目标。

想要更好的判断是否为好的分布式系统，可以看这些特性：

- **资源共享**，例如存储空间，计算能力，数据，和服务等等
- **扩展性**，从软件和硬件上增加系统的规模
- **并发性** 多个用户同时访问
- **性能** 确保当负载增加的时候，系统想要时间不会有影响
- **容错性** 尽管一些组件暂时不可用了，整个系统仍然是可用的
- **API抽象** 系统的独立组件对用户隐藏，仅仅暴露服务

有了ZooKeeper，开发者可以很轻松的实现：

- 配置管理
- 命名服务
- 分布式锁
- 集群关系操作，检测节点的加入和离开

### 分布式系统的难点

可以想象，假如一台计算机的出错概率为0.1%，那么1000台服务器的出错概率呢？一旦计算机的数量增多，出错的概率就大大的增加。

1. 多个相互独立的计算机，假设集群的配置信息在某个Master节点上，其余的节点从Master节点下载配置信息。假如Master节点挂了呢？假设Master节点是故障冗余的，但是配置信息是动态的传递给所有的其余节点的，而不是直接传过去。所有节点之间的信息如何保证一致呢？
2. 服务发现的问题，为了增加系统的可靠性，我们一般会在系统中增加更多的服务器。让其它机器知道新加入的节点在集群中的关系和服务，这个设计也需要非常周到的考虑
3. 机器数目众多，更容易出现 机器故障，软件崩溃，网络延迟，拓扑改变等等，而这些类型的错误没有规律可循，因此在分布式系统，想实现高容错性是很难的。

当然了..ZooKeeper被设计出来的目的就是解决这种类型的问题.

### ZooKeeper介绍

zookeeper实际上是yahoo开发的，用于分布式中一致性处理的框架。最初其作为研发Hadoop时的副产品。由于分布式系统中一致性处理较为困难，其他的分布式系统没有必要费劲重复造轮子，故随后的分布式系统中大量应用了zookeeper，以至于zookeeper成为了各种分布式系统的基础组件，~~其地位之重要，可想而知~~。著名的hadoop、kafka、dubbo 都是基于zookeeper而构建。

### 安装ZooKeeper

有关分布式的理论也很重要，我们放到下此再讲，首先有一个可以运行的Demo，在看理论的时候才更明白是怎么回事，因为ZooKeeper的简单易用，几分钟内就可以做出一个小demo。

1. 从ZooKeeper官网下载
    下载地址：[https://archive.apache.org/dist/zookeeper/](https://link.jianshu.com?t=https%3A%2F%2Farchive.apache.org%2Fdist%2Fzookeeper%2F)
2. 解压配置



```bash
tar -xf /usr/local/src/zookeeper-3.4.9.tar.gz -C /usr/local/src/
ln -sv /usr/local/src/zookeeper-3.4.9/ /usr/local/zookeeper
cd /usr/local/zookeeper/
```

1. 配置ZooKeeper



```csharp
vim zoo.cfg
# zoo.cfg文件中内容如下
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

- tickTime 单位为微秒，用于session注册和客户端和ZooKeeper服务的心跳周期。session超时时长最小为 tickTime的两倍
- dataDir ZooKeeper的状态存储位置，看名字就知道书数据目录。在你的系统中检查这个目录是否存在，如果不存在手动创建，并且给予可写权限。
- clientPort 客户端连接的端口。不同的服务器可以设置不同的监听端口，默认是2181

1. 启动ZooKeeper



```bash
# 这里命令写的长是为了便于知道ZooKeeper是如何使用配置文件的。
/usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo.cfg  

# 查看ZooKeeper是否运行
ps –ef | grep zookeeper 
# 也可以使用jps ，可以看到java进程中有QuorumPeerMain列出来。

# 查看ZooKeeper的状态
zkServer.sh status

# 常用的ZooKeeper用法，这个属于Linux基础的部分，就不过多说明了
./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
```

1. 使用zkCli连接ZooKeeper



```bash
/usr/local/zookeeper/bin/zkCli.sh -server localhost:2181
```

连接成功后可以使用如下命令：



![image-20200511195250031](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200511195250031.png)

### ZooKeeper集群配置

ZooKeeper的集群相对比较简单，这里不涉及过多的篇幅，但会列出大概的步骤。

1. 创建配置文件



```bash
cd /usr/local/zookeeper
touch zoo1.cfg zoo2.cfg zoo3.cfg
```



![image-20200511195320381](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200511195320381.png)

**注意** 端口不要冲突，dataDir不要相同

1. 配置数据目录与数据存放目录内容



```bash
cd /tmp/zookeeper
mkdir {zoo1,zoo2,zoo3}
echo 1 > zoo1/myid
echo 2 > zoo2/myid
echo 3 > zoo3/myid
```

这里的myid文件中一定要对应上面配置文件中server.[id]的数字，不然ZooKeeper启动会出错。

1. 启动ZooKeeper



```bash
zkServer.sh start /usr/local/zookeeper/conf/zoo1.cfg
zkServer.sh start /usr/local/zookeeper/conf/zoo2.cfg
zkServer.sh start /usr/local/zookeeper/conf/zoo3.cfg
```

1. 查看效果

使用ps -ef | grep zoo可以看到有三个zookeeper启动起来了。
 连接ZooKeeper



```bash
# 192.168.8.250是ZooKeeper服务器的地址
zkCli.sh -server 192.168.8.250:2181,192.168.8.250:2182,192.168.8.250:2183
```

![image-20200511195406435](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200511195406435.png)

集群连接效果

可以看到连接成功，也就是集群配置成功了。

### 最后

这篇文章，主要介绍了一下什么是分布式系统和分布式系统的的基本特性，然后又做了ZooKeeper的简单Demo，希望帮助大家入门ZooKeeper。

