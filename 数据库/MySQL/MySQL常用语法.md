在开发中你可以不会那些高级的api但是你一定不能不会写sql

作为一个资深的CRUD程序猿

这里写几个简单的CRUD语句给大家看看什么是CRUD程序猿

## 插入

```
语法：

 insert into table_name values  ( value1, value2, ...*)
 insert into table_name  ( column1, column2, ... )  values  ( value1, value2, ... )

示例：新增注册用户

sql insert into `users`  values (1, 'ken', '666',  curdate ())

/* 指定列插入 */

 insert into  `users` (`name`, `avatar`)  values  ('bill', '666')
```



## 修改

```
语法：

update table_name set column=new_value where condition
update table_name set column1=new_value1,column2=new_value2,... where  condition

示例：

sql  update `users`  set  `regtime`= curdate ()  where  `regtime`  is  null

/* 一次修改多列 */

 update  `users`  set  `name`='steven',`avatar`='666'  where  `id`=1
```



## 删除

```
语法：delete from table_name where condition 
```



## 查询

### 1-查询所有列

```
select * from users;
```

### 2-查询指定列

```
select id,name from users;
```

### 3-查询不重复记录

```
select distinct name,avatar from users;
```

### 4-限制查询行数(分页)

select id,name from users limit 2;

### 5-查询从指定偏移(第一行为偏移为0)开始的几行

```
select id,name from users limit 2,3;
```

### 6-排序

#### 正序

```
select distinct name from users order by name asc limit 3;
```

#### 倒序

```
 select id,name from users order by id desc limit 3;
```

### 7-分组

```
select city, count(name) as num_of_user from users group by city;
```

### 8-多表关联查询

#### join

```
select `users`.`name` as `user_name`, `orders`.`title` as `order_title` from `users`, `orders` where `orders`.`user_id`=`users`.`id`;
```

#### inner join

```
select `users`.`name` as `user_name`, `orders`.`title` as `order_title` from `users` inner join `orders` on `orders`.`user_id`=`users`.`id`;
```

内部连接。效果与 join 一样 , 但用法不同，join 使用 where ，inner join 使用 on 。

#### left join

左连接。返回**左表**所有行，即使**右表**中没有匹配的行，不匹配的用 NULL 填充。

```
select `users`.`name` as `user_name`, `orders`.`title` as `order_title` from `users` left join `orders` on `orders`.`user_id`=`users`.`id`;
```

#### right join

右连接。和 left join 正好相反，会返回**右表**所有行，即使**左表**中没有匹配的行，不匹配的用 NULL 填充。

```
select `groups`.`title` as `group_title`, `users`.`name` as `user_name` from `groups` right join `users` on `users`.`group_id`=`groups`.`id`;
```

### union

union 用于合并两个或多个查询结果，合并的查询结果必须具有相同数量的列，并且列拥有形似的数据类型，同时列的顺序相同。

对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；

```
 (select `id`, `title` from `groups`) union (select `id`, `title` from `orders`);
```

union All 对两个结果集进行并集操作，包括重复行，不进行排序；

```
 (select `id`, `title` from `groups`) union all (select `id`, `title` from `orders`);
```

