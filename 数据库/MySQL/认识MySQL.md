## 架构图

![image-20200803093936868](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803093936868.png)

### 详细架构

![image-20200803094010113](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803094010113.png)

![image-20200803094107223](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803094107223.png)

![image-20200803094124322](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803094124322.png)

解析整个结构 从流程来说

#### 初始化模块

加载默认参数, buffer,chach等.等一系列初始化

#### 连接管理模块

监听soket请求 拿到请求分发到

#### 连接进程模块(连接池概念)

有连接空闲就用这个空闲连接,没有就创建连接-->再进行分发到用户模块

#### 用户模块

看用户是否有权限操作

#### 命令分发器

##### 

#### 查询缓存模块(只针对查询)

类似map  把sql语句和参数 做key  结果集做value

#### 日志模块

log记录 binlog  undo  ....

#### 命令解析器

见上图



#### 核心API

类似java中的jvm 内存管理  IO  

