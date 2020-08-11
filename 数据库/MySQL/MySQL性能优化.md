首先先明白

## 影响性能的因数(宏观层面)

### 1-人为因数-需求

我们可以从需求出发,  数据是否要实时?准实时?还是可以有误差

### 2-程序员因数-面向对象

举个栗子

![image-20200803111549779](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803111549779.png)

如果按面向对象开发 方案一更优 但是要进行11次IO

但是通过IO对比  方案2只需要进行2次IO 明显更优 

结论:不能太面向对象

### 3-Cache

加缓存和消息队列 不要让所有请求都打到数据库里面

### 4-对可扩展过度追求

比如字段冗余

### 5-表范式

过于追求范式

### 6-应用场景

#### OLTP	On-Line Transaction Processioning

![image-20200803112859914](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803112859914.png)

#### OLAP	On-Line Analysis Processing(数据仓库)

![image-20200803112721547](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803112721547.png)

## 如何提升性能

### 索引

简单来说就是 高效获取数据

衡量一个索引好不好的标准就是 IO渐进复杂度

也就是说当数据越来越多的时候 我们的索引是不是还是那么高效

### 数据结构

Hash 无法做范围查询

FullText(全文搜索)

R-Tree(空间索引)

### MYISAM

![image-20200803220959051](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803220959051.png)

  MYI叶子节点存储的是MYD文件的地址

这种叫非聚集索引

### INNODB

![image-20200803224637897](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200803224637897.png)

特点是首先会以主键作为一个索引,叶子节点存储的是我们的真实数据 而不是内存地址



### 索引总结

![image-20200810161228024](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200810161228024.png)



好处就是pros

坏处就是cons

尽可能少创建索引

如果必要,请创建联合索引,然后在sql中按最左原则查询

联合索引还有一个好处就是 对于第二列是可以做到一个排序的作用

尽可能创建自增型索引,因为无序插入索引 时候 不知道这个索引是增加在左边还是右边

如果是自增型索引的话 新增的数据永远会在右边,物理连续性会更高 且对树的结构不需要太大改动



## 锁

### 行锁

![image-20200810162855337](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200810162855337.png)

### 表锁

![image-20200810162910065](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200810162910065.png)

### 锁总结

事务不宜做的过大

-- 表级锁的争用状态变量

show status like 'table%';

-- 行级锁争用状态变量

show status like 'innodb_row_lock%';





### 优化总结

![image-20200810174430337](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200810174430337.png)



我们优化的目标是尽可能减少JOIN中Nested Loop的循环次数，以此保证：永远用小结果集驱动大结果集

使用 加上 explain 可以根据对应的执行效率进行优化

explain简单介绍

| id          | id一致表示在同一个sql语句中                                  |
| ----------- | ------------------------------------------------------------ |
| select_type | 表示这个查询是一个什么查询(子查询,union查询==)               |
| table       | 作用于哪个表 (注意:第一个是主驱动表  第二个是被驱动表)       |
| partition   | 分区                                                         |
| type        | 告诉我们对表的访问方式(是否走没走索引)                       |
| possible    | 该查询可以利用的索引.如果没有任何索引可以使用,就会显示成null |
| key         | 使用的索引                                                   |
| key_len     | 索引长度(优化要选len短一点)                                  |
| ref         | 过滤条件以什么形式                                           |
| rows        | 本次扫描的数据量是多少                                       |
| extra       |                                                              |



### JOIN 底层实现和优化

![image-20200811100844392](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811100844392.png)

-- show variables like 'join_%';

查看join buffer 的大小

如何优化

![image-20200811100936692](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811100936692.png)

### OrderBy 底层实现和优化

![image-20200811101111892](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811101111892.png)

有索引就直接走索引排序  因为b+tree的叶子节点都是相互排序指向的

没有索引就是会把数据放到一个sort buffer中 

然后根据排序字段+指针指向(也就是地址指向)最后根据排序结果将结果写到result中

这种orderby实现实际上做了2次IO  1和2

还有一种只做了一次IO的实现

![image-20200811101955155](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811101955155.png)

这种是将你要返回的表的数据全部放到buffer中  然后再根据你需要排序的字段 进行排序 指针直接指向内存 然后将排好的数据直接返回到result中  典型的**空间换时间**

这2种排序是mysq自动选择的  mysql会根据你当前的buffer来选择最优的orderby

这也就是为什么说只取出自己需要的字段也是一种优化  **建议不要使用select ***

如何优化

![image-20200811101126890](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811101126890.png)



### Group By

group by的前提是order by 所以order by的优化对Group By也生效



### Distinct

Distinct 又是基于Group By的结果来进行的



### Limit

分析为什么慢? 下面这语句

SELECT * FROM	user	limit 10000,10;

数据库底层会取 10010 条数据

 如何解决?

如果id是自增的话就可以这样

SELECT * FROM user where id> 10000 limit 10;

## 总结

![image-20200811103645795](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200811103645795.png)

优化要从整体的一个分析来进行

首先是宏观层面

数据库层面

最后是sql语句层面

**任何数据库层面的优化都抵不上应用系统的优化**

推荐文章

https://tech.meituan.com/2014/06/30/mysql-index.html