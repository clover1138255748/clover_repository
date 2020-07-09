这里日志可能有点问题 只打印出一个nullpointterException

没有其他信息

![image-20200703103721932](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703103721932.png)

这时候启动arths

选择进程8 就是我们bill模块

![image-20200703103828840](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703103828840.png)

这里直接找到报错方法 然后watch一下 参数返回值 异常都会打印出来

![image-20200703104018704](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703104018704.png)

![image-20200703104217879](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703104217879.png)

猜想  磅单上车牌号码填写错误 导致车牌找不到车 

getid就报空指针异常 

这里加个判断  

![image-20200703104528362](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703104528362.png)

考虑到这个异常   那这磅单必然走磅单异常 

磅单异常后修改车牌号时候也要去查下车牌加上truck_id

![image-20200703104923068](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200703104923068.png)

不然 待会这磅单去手工配对时候和托运单的truck_id对不上