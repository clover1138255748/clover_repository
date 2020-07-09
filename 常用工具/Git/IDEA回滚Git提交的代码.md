今天一个朋友问我GIt提交错了分支怎么回滚

????

我觉得还是写一个详细教程来帮助大家

这里用的工具是IDEA  GitLab

首先我们先提交一个版本 假设这个版本你提交错了分支

![image-20200630151236592](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630151236592.png)

首先我们先看看你这个版本前面提交的版本是什么 这里是委托单受理修改

现在idea中找到log

![image-20200630151804075](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630151804075.png)

版本号就是这个

f150c4680923b678114009f56f7899eaa3e116b1

然后我们就准备回滚它

![image-20200630153958328](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630153958328.png)

这里看到我们的代码已经到这里了 

![image-20200630154116092](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630154116092.png)

我们直接回滚

![image-20200630154158562](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630154158562.png)



当我们再登录GitLab的时候就发现历史记录都没有了

![image-20200630154253891](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200630154253891.png)