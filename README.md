# clover_repository


本文分为以下几个大块来一步步梳理我们平常所接触到的JAVA技术



## 一   JAVA基础知识

[JAVA基础知识汇总](JAVA基础知识/Java基础知识.md)

[枚举的使用](JAVA基础知识/枚举的使用.md)

[序列化反序列化原理和知识点](JAVA基础知识/序列化反序列化原理和知识点.md)

### 	List

​		[ArrayList源码解析](JAVA基础知识/List/ArrayList源码解析.md)		

​		[CopyOnWriteArrayList源码解析](AVA基础知识/List/CopyOnWriteArrayList源码解析.md)

​		[LinkedList源码解析](JAVA基础知识/List/LinkedList源码解析.md)

### 	Map

​		[HashMap简介](JAVA基础知识/Map/HashMap.md)

​		[HashMap源码解析](JAVA基础知识/Map/HashMap源码解析.md)

​		[手写LRU缓存](JAVA基础知识/Map/手写LRU缓存.md)

## 二   JAVA多线程知识

[AQS的原理以及应用](JAVA多线程/AQS的原理以及应用.md)

[详解Volatile关键字](JAVA多线程/详解Volatile关键字.md)

[乐观锁与悲观锁](JAVA多线程/乐观锁与悲观锁.md)

[synchronized的实现原理](JAVA多线程/synchronized的实现原理.md)

[CAS底层实现原理](JAVA多线程/CAS底层实现原理.md)

[ThreadLocal](JAVA多线程/ThreadLocal.md)

## 三   JAVA常用框架

### 	SpringBoot

​			[SpringBoot是什么](常用框架/SpringBoot/SpringBoot是什么.md)

​			[27个SpringBoot核心注解](常用框架/SpringBoot/27个SpringBoot核心注解.md)

​			[SpringBoot是如何进行自动装配的](常用框架/SpringBoot/SpringBoot是如何进行自动装配的.md)

​			[SpringBoot部分关键应用配置项说明](常用框架/SpringBoot/SpringBoot部分关键应用配置项说明.md)

### 	SpringMVC

### 	Spring

​			[Spring入门指南](常用框架/Spring/Spring入门指南.md)

​			[IOC和AOP的基础理念](常用框架/Spring/IOC和AOP的基础理念.md)

​			[Spring如何解决循环依赖](常用框架/Spring/Spring如何解决循环依赖.md)

#### 		IOC

​			[IOC理解](常用框架/Spring/IOC/IOC理解.md)

​			[IOC原理分析](常用框架/Spring/IOC/IOC原理分析.md)

#### 		AOP

​			[AOP实现自定义注解(代码版)](常用框架/Spring/AOP/AOP实现自定义注解.md)

​			[AOP实现请求接口插入日志](常用框架/Spring/AOP/AOP实现请求接口插入日志.md)

### 	MyBatis

## 四   JAVA数据库知识

### 	MySQL

​			[认识MySQL](数据库/MySQL/认识MySQL.md)

​			[MySQL重要知识点](数据库/MySQL/MySQL重要知识点.md)

​			[深入浅出数据库索引原理](数据库/MySQL/深入浅出数据库索引原理.md)

​			[一条SQL语句在MySQL中如何执行的](数据库/MySQL/一条SQL语句在MySQL中如何执行的.md)

​			[MySQL常用函数](数据库/MySQL/MySQL常用函数.md)

​			[MySQL常用语法](数据库/MySQL/MySQL常用语法.md)			

​			[MySQL高性能优化规范建议](数据库/MySQL/MySQL高性能优化规范建议.md)

​			[MySQL性能优化](数据库/MySQL/MySQL性能优化.md)

​			[MySQL几种常用的log](数据库/MySQL/MySQL几种常用的log.md)

​			[MySQL之复杂报表(车辆活跃表)](数据库/MySQL/MySQL之复杂报表(车辆活跃表).md)

### 	Redis  

​			[彻底理解 IO多路复用](数据库/Redis/彻底理解 IO多路复用.md)

​			[Redis入门指南](数据库/Redis/Redis入门指南.md)

​			[Redis的线程模型(为啥单线程效率这么高)](数据库/Redis/Redis的线程模型(为啥单线程效率这么高).md)

​			[Redis的主从复制](数据库/Redis/RedisRedis的主从复制.md)

​			[Redis的持久化原理](数据库/Redis/RedisRedis的持久化原理.md)

​			[Redis是怎么保证高并发高可用的](数据库/Redis/Redis是怎么保证高并发高可用的.md)

​			[Redis的过期策略以及内存淘汰机制](数据库/Redis/RedisRedis的过期策略以及内存淘汰机制.md)

​			[Redis实现一个消息队列](数据库/Redis/Redis实现一个消息队列.md)

### 	Elasticsearch

​			[ES的分布式架构原理](数据库/Elasticsearch/ES的分布式架构原理.md)

​			[ES写入数据的工作原理](数据库/Elasticsearch/ES写入数据的工作原理.md)

## 五   JAVA高性能网站架构

[分布式锁的使用场景](高性能网站架构/分布式锁的使用场景.md)

[基于Redis实现分布式锁](高性能网站架构/基于Redis实现分布式锁.md)

[基于zookeeper实现分布式锁的几种方式](高性能网站架构/基于zookeeper实现分布式锁的几种方式.md)

[redis与zokeeper分布式锁的优缺点](高性能网站架构/redis与zokeeper分布式锁的优缺点.md)

### 	SpringCloud

​			[SpringCloud底层原理](高性能网站架构/SpringCloud/SpringCloud底层原理.md)

​			[SpringCloud-Hystrix-Feign处理降级](高性能网站架构/SpringCloud/SpringCloud-Hystrix-Feign处理降级.md)

​			[Spring Cloud Zuul过滤器介绍及使用](高性能网站架构/SpringCloud/Spring Cloud Zuul过滤器介绍及使用.md)

### 	Dobbo

​			[Dubbo底层原理](高性能网站架构/Dobbo/Dubbo底层原理.md)

### 	MQ

​			[为什么要引入消息中间件](高性能网站架构/MQ/为什么要引入消息中间件.md)

​			[如何保证消息队列的高可用](高性能网站架构/MQ/如何保证消息队列的高可用.md)

​			[如何处理消息丢失的问题](高性能网站架构/MQ/如何处理消息丢失的问题.md)

#### 			**RabbitMQ**

​						[一文搞懂 RabbitMQ 的重要概念](高性能网站架构/MQ/RabbitMQ/一文搞懂 RabbitMQ 的重要概念.md)

### 	Seata

​			[Seata入门指南](高性能网站架构/Seata/Seata入门指南.md)

​			[Seata踩坑指南](高性能网站架构/Seata/Seata踩坑指南.md)

### 	Zookeeper

​			[ZooKeeper入门指南](高性能网站架构/Zookeeper/ZooKeeper入门指南.md)

## 六   JAVA常用工具

### 	Arthas

​			[Arthas介绍以及基础用法](常用工具/Arthas/Arthas介绍以及基础用法.md)

​			[Arthas(阿尔萨斯)源码原理分析](常用工具/Arthas/Arthas(阿尔萨斯)源码原理分析.md)

​			[Arthas实践--jad/mc/redefine线上热更新一条龙](常用工具/Arthas/Arthas实践--jad/mc/redefine线上热更新一条龙.md)

### 	Docker

### 	[Git](常用工具/Git/README.md)

### 	[SVN](常用工具/SVN/README.md)

### 	Maven

​			[Maven基础使用知识点](常用工具/Maven/Maven基础使用知识点.md)

### 	[IDEA](常用工具/IDEA/README.md)

[Jmeter入门指南(性能压测工具)](常用工具/Jmeter入门指南.md)

## 七   JAVA设计模式

[设计模式的由来](设计模式/设计模式的由来.md)

[设计模式简介](设计模式/设计模式简介.md)

### 	创建型模式

​			[工厂模式]()

​			[抽象工厂模式]()

​			[单例模式]()

​				[如何优雅的写一个单例模式](设计模式/单例模式/如何优雅的写一个单例模式.md)

​			[建造者模式]()

​			[原型模式]()

### 	结构型模式

​			[适配器模式]()

​			[桥接模式]()

​			[过滤器模式]()

​			[组合模式]()

​			[装饰器模式]()

​			[外观模式]()

​			[享元模式]()

​			[代理模式]()

### 	行为型模式

​			[责任链模式]()

​			[命令模式]()

​			[解释器模式]()

​			[迭代器模式]()

​			[中介者模式]()

​			[备忘录模式]()

​			[观察者模式]()

​			[状态模式]()

​			[空对象模式]()

​			[策略模式]()

​			[模板方法模式]()

​			[访问者模式]()

## 八   程序人生

### 	面试

#### 			面经

​				[阿里一面](程序人生/面试/软通面试.md)	

​				[阿里二面](程序人生/面试/阿里面经.md)	

​				[阿里笔试](程序人生/面试/阿里笔试.md)	

​				[恒生面试](程序人生/面试/恒生面试.md)

​			[多线程面试答疑](程序人生/面试/面经/多线程面试答疑.md)

#### 			面试简历

​				[自我介绍](程序人生/面试/面试简历/面试自我介绍篇.md)

### 	线上事故

​			[arthas排除错误之磅单车牌错误问题](程序人生/线上事故/arthas排除错误之磅单车牌错误问题.md)

### 	项目搭建

​			[分析云项目](程序人生/项目搭建/分析云项目.md)

### 	项目方案

​			[大屏数据展示需求案例分析](程序人生/项目方案/大屏数据展示需求案例分析.md)

​			[大屏数据展示需求字段对应](程序人生/项目方案/大屏数据展示需求字段对应.md)

​			[大屏数据展示需求字段对应](程序人生/项目方案/大屏数据展示需求字段对应.md)

### 	疑难BUG

## 九   其他

### 	JDK

​			[JDK1.8学习笔记](其他/JDK/JDK1.8学习笔记.md)

​			[Optional的实战用法](其他/JDK/Optional的实战用法.md)

​			[Java8LocalDate](其他/JDK/Java8LocalDate.md)

### 	JVM

​			[JVM类加载机制](其他/JVM/JVM类加载机制.md)

​			[JVM类加载过程](其他/JVM/JVM类加载过程.md)

​			[JVM内存模型](其他/JVM/JVM内存模型.md)

​			[JVM垃圾回收器和算法](其他/JVM/JVM垃圾回收器和算法.md)

### 	Netty

​			[认识Netty](其他/Netty/认识Netty.md)

[史上最全的Java命名规范参考](其他/史上最全的Java命名规范参考.md)

[Java处理Exception的9个最佳实践](其他/Java处理Exception的9个最佳实践.md)

## 特别篇 数据结构

​			[什么是数据结构](数据结构/什么是数据结构.md)

​			[算法时间复杂度和空间复杂度的计算](数据结构/算法时间复杂度和空间复杂度的计算.md)			

​			[数据结构-堆](数据结构/数据结构-堆.md)

​			[数据结构-队列](数据结构/数据结构-队列.md)

​			[数据结构-二叉树](数据结构/数据结构-二叉树.md)

​			[数据结构-哈希表](数据结构/数据结构-哈希表.md)

​			[数据结构-红黑树](数据结构/数据结构-红黑树.md)			

​			[数据结构-链表](数据结构/数据结构-链表.md)

​			[数据结构-数组](数据结构/数据结构-数组.md)

​			[数据结构-图](数据结构/数据结构-图.md)

​			[数据结构-栈](数据结构/数据结构-栈.md)

​			[数据结构-2-3-4数](数据结构/数据结构-2-3-4数.md)