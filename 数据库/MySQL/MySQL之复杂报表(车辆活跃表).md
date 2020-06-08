# 需求

![image-20200605165659218](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200605165659218.png)

BOSS发话 明天要用

最讨厌产品的需求就是规则和老系统一样

我giao

你这是新系统表结构设计逻辑什么都不一样  你怎么叫我弄的一样咯

乜办法  BOSS发话了  一天就一天吧  硬着头皮上

一步一步来吧

## 1-需求分析

老系统是这样的

![image-20200605170101629](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200605170101629.png)

现在要做成这样

![image-20200605170157184](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200605170157184.png)

少了一个资料完善



然后我简单整理了一下逻辑

发现一个比较比较致命的问题

就是老系统的累计是塔喵的有一个专门的定时任务每天去捞取统计好的放在一张指定的表内的  我新系统哪里有这玩意

![image-20200605170546071](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200605170546071.png)

如果按老系统逻辑走那估计今天是搞不完了



## 2-需求实现

![image-20200606091359104](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200606091359104.png)

### 1-1累计实现

这报表难点其实就在这里这个累计

新增的sql知道查哪张表很简单 只要在查询日期内做一个分组就知道每天新增多少 举个注册数例子

```
-- 注册数
SELECT
		DATE_FORMAT(t1.gmt_create , '%Y-%m-%d') as gmt,
		count(*) num
	from
		cif.login t1
	inner join cif.`identity` t2 on
		t1.user_id = t2.user_id
	where
		1 = 1
		and t1.gmt_create >= '2020-05-20'
		and t1.gmt_create <= '2020-05-21'
		and t1.user_role_type = 'TRUCK_OWNER'
	GROUP by
		gmt
```

入驻数同理

因为累计要进行一个把前一天的也统计进去 所以就比较难搞 如果用写代码的方式也是把日期分成一份一份 然后for循环查数据

如果用户有30天那就是查30次数据库 加上入驻数那就是60次  那肯定是不行的!

后来我查了下资料有没有查这种MySQL实现累计求和操作

还真有

我把我实现后的sql贴出来

```
SELECT
		qq.days days,
		sum(ww.num) as total
	FROM
		(
		------------------------------------------------
		SELECT
			DATE_FORMAT(t2.gmt_create , '%Y-%m-%d') as days,
			count(*) as num
		from
			cif.login t1
		inner join cif.`identity` t2 on
			t1.user_id = t2.user_id
		where
			1 = 1
			and t2.gmt_create >= '2020-06-01'
			and t2.gmt_create <= '2020-06-05'
			and t1.user_role_type = 'TRUCK_OWNER'
		GROUP by
			days 
		------------------------------------------------
			) qq
	join (
	------------------------------------------------------
		SELECT
			DATE_FORMAT(t2.gmt_create , '%Y-%m-%d') as days,
			count(*) as num
		from
			cif.login t1
		inner join cif.`identity` t2 on
			t1.user_id = t2.user_id
		where
			1 = 1
			and t2.gmt_create >= '2020-06-01'
			and t2.gmt_create <= '2020-06-05'
			and t1.user_role_type = 'TRUCK_OWNER'
		GROUP by
			days
     ------------------------------------------------ 
      )ww on
		qq.days >= ww.days
	GROUP by
		qq.days 
```

sql是比较复杂的

我用分割新把代码割出来就就很清晰

可以看出 2个分割线的sql是一模一样的

唯一不同的是qq.days >= ww.days

一般来说on后面都是把2段sql连起来 

但是这个sql意思就是统计每次都是加上前一天的

这里再说说 如果我只想累计本月的 假如说5月我累计到30号 6月我要重新开始累计怎么操作呢很简单在

qq.days >= ww.days 后面贴上

AND DATE_FORMAT( qq.days, '%Y-%m' ) = DATE_FORMAT( ww.days, '%Y-%m' )

这就很nice

只能说学到了

下面贴结果 拿这个06-02举例子  02新增数是37就对了.我们去查下



![image-20200606092911513](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200606092911513.png)

![image-20200606093100448](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200606093100448.png)

后面就不一一验证了 反正就是这么个意思

然后把注册数和累计注册数放在一起用日期连接起来,sql就是下面这样

```
SELECT 
	www.days,
	qqq.num,
	www.total
from
	(
	---------------------------------------------
	SELECT
		DATE_FORMAT(t1.gmt_create , '%Y-%m-%d') as gmt,
		count(*) num
	from
		cif.login t1
	inner join cif.`identity` t2 on
		t1.user_id = t2.user_id
	where
		1 = 1
		and t1.gmt_create >= '2020-05-20'
		and t1.gmt_create <= '2020-05-21'
		and t1.user_role_type = 'TRUCK_OWNER'
	GROUP by
		gmt
		--------------------------------------------------
		) qqq
inner join (
----------------------------------------------------
	SELECT
		qq.days days,
		sum(ww.num) as total
	FROM
		(
		--------------------------------------------
		SELECT
			DATE_FORMAT(t2.gmt_create , '%Y-%m-%d') as days,
			count(*) as num
		from
			cif.login t1
		inner join cif.`identity` t2 on
			t1.user_id = t2.user_id
		where
			1 = 1
			and t2.gmt_create >= '2020-06-01'
			and t2.gmt_create <= '2020-06-05'
			and t1.user_role_type = 'TRUCK_OWNER'
		GROUP by
			days ) qq
	join (
		SELECT
			DATE_FORMAT(t2.gmt_create , '%Y-%m-%d') as days,
			count(*) as num
		from
			cif.login t1
		inner join cif.`identity` t2 on
			t1.user_id = t2.user_id
		where
			1 = 1
			and t2.gmt_create >= '2020-06-01'
			and t2.gmt_create <= '2020-06-05'
			and t1.user_role_type = 'TRUCK_OWNER'
		GROUP by
			days )ww on
		qq.days >= ww.days
		------------------------------------------------------------------------------
	GROUP by
		qq.days ) www on
	qqq.gmt = www.days
	
```

可能有点复杂,但是拆分一下还是很简单的

然后我们只要查到开始日期之前注册了多少车+累计的车的话就实现了这个累计求和操作

```
SELECT
		count(*) as num 
	from
		cif.login t1
	inner join cif.`identity` t2 on
		t1.user_id = t2.user_id
	where
		1 = 1
		and t2.gmt_create < '${beginDate}'
		and t1.user_role_type = 'TRUCK_OWNER'
```

车辆入驻同理

### 1-2活跃实现

#### 日活跃

简单就是根据日期分组然后count(DISTINCT xx)完事

```
-- 日活跃车辆  2983
 SELECT
 	 count(DISTINCT driver_truck_id) countNum,
 	 DATE_FORMAT(receipt_time , '%Y-%m-%d') as days
	
from
	bill.waybill w
where 1=1 
	and receipt_time >= '${beginDate}'
	and receipt_time <= '${endDate}'
group by days

```

#### 周活跃

之前被这个周活跃难住了 

因为这个周活跃要显示在每周的格子内

![image-20200606094455835](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200606094455835.png)

后来用了这个函数搞定了

就是这个WEEK函数 我们看下官方介绍

![image-20200606094706086](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200606094706086.png)

代码如下

```
SELECT
	DATE_FORMAT(ct.receipt_time, '%Y-%m-%d') days,
	WEEK (ct.receipt_time,
	1) weekgroups,
	IFNULL(COUNT(DISTINCT(ct.driver_truck_id)),
	0) countNum
FROM
	bill.waybill ct
WHERE
	1 = 1
	and receipt_time >= '${beginDate}'
	and receipt_time <= '${endDate}'
GROUP BY
	weekgroups
```

这个 weekgroups就是关键

#### 月活跃

```
SELECT
	 count(DISTINCT driver_truck_id) countNum
from
	bill.waybill w
where
	1 = 1
	and receipt_time >= '${beginDate}'
	and receipt_time <= concat('${endDate}','23:59:59')
```

