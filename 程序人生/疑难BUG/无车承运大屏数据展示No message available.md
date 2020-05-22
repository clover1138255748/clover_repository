## 业务场景

![image-20200521150043537](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200521150043537.png)

因为Hystrix断路器影响 我把返回的参数做了一个分类 user bill finance  这是3个不同的微服务

然后我在调用时候确出现了

![image-20200521151243314](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200521151243314.png)

这种BUG  

先检查是否调用