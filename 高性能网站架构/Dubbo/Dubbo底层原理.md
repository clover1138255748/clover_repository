![dubbo-framework-principle](https://gitee.com/cdx_dayshow/picBed/raw/master/img/dubbo-framework-principle.png)

提供接口

服务注册中心：

\###消费者

动态代理：Proxy
负载均衡：Cluster，负载均衡，故障转移
注册中心：Registry
通信协议：Protocol，filter机制，http、rmi、dubbo等协议
http、rmi、dubbo
比如说，我现在其实想要调用的是，DemoService里的sayHello接口

你的请求用什么样的方式来组织发送过去呢？以一个什么样的格式来发送你的请求？

http，/demoService/sayHello?name=leo rmi，另外一种样子 dubbo，另外一种样子，interface=demoService|method=sayHello|params=name:leo

信息交换：Exchange，Request和Response

对于你的协议的格式组织好的请求数据，需要进行一个封装，Request

网络通信：Transport，netty、mina
序列化：封装好的请求如何序列化成二进制数组，通过netty/mina发送出去
提供者

网络通信：Transport，基于netty/mi