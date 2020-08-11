本文我们来介绍下在使用Feign来做服务调用的情况下怎么通过Hystrix实现服务降级处理。

# Feign实现服务降级

### 1-首先你要有个springCloud的项目

![image-20200604144017408](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604144017408.png)

### 2.修改配置文件

  feign中默认是关闭对Hystrix的支持的，我们需要放开设置

![image-20200604144232027](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604144232027.png)

### 3.业务处理

  业务层接口中我们不需要再继承自service中的父接口了，直接在接口中声明一模一样的方法接口

![image-20200604163913144](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604163913144.png)

这里Api定义一个fallback=xxx.class

这里的WorkOrderApiFallBack.class就是你降级处理的策略

![image-20200604164101279](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164101279.png)



这里怎么理解呢 ? 大概就是你这个WorkOrderApiFallBack实现这个Api接口

一般情况不走这个类的返回值的 但是你要是那边正常走的流程出现了错误就会触发这个降级策略

我们先定义一个错误

![image-20200604164932328](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164932328.png)

然后这里我们测试一下

![image-20200604164336166](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164336166.png)

![image-20200604164407396](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164407396.png)

这里我们发现触发了降级

这是成功版本

这里踩了一个坑就是我们这边系统就算你报错了 我们是返回状态码200

然后是这样的一个结果

![image-20200604164531249](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164531249.png)

其实我们是报错了 但是我们还是返回200

所以不会触发降级

后来我们在GlobalExceptionHandler之类把这个注释掉了  不会处理异常 异常就返回500 然后就降级成功了

![image-20200604164630538](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200604164630538.png)

但是leader说APP那边需要返回200 不然会有问题 所以这个降级算是失败了  只有超时情况才会触发降级