#### 概念：

相当于java中的方法，将一组逻辑语句封装在方法体中，对外暴露方法名
1）隐藏了实现细节   2）提高代码的可重用性

#### 使用：

select 函数名(实参列表)【from 表】    【】中内容可省略

### 正文： 

##### 字符函数：

- length： 获取字节个数（utf-8 一个汉字为3个字节，gbk为2个字节）

  ```sql
  SELECT LENGTH('cbuc')    # 输出 4
  SELECT LENGTH('蔡不菜cbuc')   # 输出13
  复制代码
  ```

- concat： 拼接字符串

  ```sql
  SELECT CONCAT('C','_','BUC')   # 输出 C_BUC
  复制代码
  ```

- upper：将字母变成大写

  ```sql
  SELECT UPPER('cbuc')    # 输出 CBUC
  复制代码
  ```

- lower：将字母变成小写

  ```sql
  SELECT LOWER('CBUC')   # 输出 cbuc
  复制代码
  ```

- substr / substring：裁剪字符串
  该方法进行了重构，

  ![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/5/31/1726af70bb11180b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```sql
substr(str,pos)       # str:要裁剪的字符串 ， pos:要裁剪的长度
substr(str,pos,len)   # str:要裁剪的字符串 , pos/len:从哪个位置开始裁剪几位
# substring同理
复制代码
```

- instr：返回子串第一次出现的索引，如果没有则返回0

  ```sql
  SELECT INSTR('蔡不菜','蔡')        # 输出 1 （mysql是从1开始算位数）
  复制代码
  ```

- trim：字符串去【字符】

  ```sql
  SELECT TRIM('  cbuc  ')                 # 输出 cbuc
  SELECT TRIM('a' from 'aaaacbucaaaa')    #输出 cbuc
  复制代码
  ```

- lpad：用指定字符实现左填充指定长度

  ```sql
  SELECT LPAD('cbuc',6,'*')            # 输出 **cbuc
  复制代码
  ```

- rpad：用指定字符实现右填充指定长度

  ```sql
  SELECT LPAD('cbuc',6,'*')            # 输出 cbuc**
  复制代码
  ```

- replace 替换

  ```sql
  SELECT REPLACE('小菜爱睡觉','睡觉','吃饭')        # 输出 小菜爱吃饭
  复制代码
  ```

##### 数学函数

- round：四舍五入

  ```sql
  SELECT round(1.5)        # 输出  2
  SELECT round(-1.5)        # 输出 -2 该四舍五入计算方式为：绝对值四舍五入加负号
  复制代码
  ```

- ceil：向上取整,返回>=该参数的最小整数

  ```sql
  SELECT CEIL(1.5);        # 输出  2
  SELECT CEIL(-1.5);        # 输出 -1
  复制代码
  ```

- floor：向下取整，返回<=该参数的最大整数

  ```sql
  SELECT CEIL(1.5);        # 输出  1
  SELECT CEIL(-1.5);        # 输出 -2
  复制代码
  ```

- truncate：截断

  ```sql
  SELECT TRUNCATE(3.1415926,2);        # 输出 3.14
  复制代码
  ```

- mod：取余

  ```sql
  SELECT MOD(10,3);        # 输出 1
  SELECT MOD(10,-3);        # 输出 1
  复制代码
  ```

##### 日期函数

- now：返回当前系统日期+时间

  ```sql
  SELECT NOW()               # 输出 2020-02-16 11:43:21
  复制代码
  ```

- curdate：返回当前系统日期，不包含时间

  ```sql
  SELECT CURDATE()        # 输出 2020-02-16
  复制代码
  ```

- curtime：返回当前时间，不包含日期

  ```sql
  SELECT CURTIME()        # 输出 11:45:35
  复制代码
  ```

- year/month/day 可以获取指定的部分，年、月、日、小时、分钟、秒

  ```sql
  SELECT YEAR(NOW())        # 输出 2020   其他用法一致
  复制代码
  ```

- str_to_date：将字符通过指定的格式转换成日期

  ```sql
  SELECT STR_TO_DATE('02-17 2020','%c-%d %Y')      # 输出 2020-02-17
  复制代码
  ```

- date_format：将日期转换成字符

  ```sql
  SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日')        # 输出 2020年02月17日
  复制代码
  ```

- datediff：两个日期天数之差

  ```sql
  SELECT DATEDIFF(NOW(),'2020-02-12')           # 输出    5
  复制代码
  ```

##### 其他函数

- VERSION：查看mysql 版本

  ```sql
  SELECT VERSION();           # 输出 5.7.17
  复制代码
  ```

- DATABASE：查看当前数据库

  ```sql
  SELECT DATABASE()          # 输出 cbuc_datebase
  复制代码
  ```

- USER：查看当前用户

  ```sql
  SELECT USER()               # 输出 root@localhost
  复制代码
  ```

##### 流程控制函数

- if 函数： 类似三目运算

  ```sql
  SELECT IF(10<5,'大','小')        # 输出 大
  复制代码
  ```

- case函数：case 有两种用法

  1. switch case 的效果

  ```sql
  case 要判断的字段或表达式
  when 常量1 then 要显示的值1或语句1;
  when 常量2 then 要显示的值2或语句2;
  ...
  else 要显示的值n或语句n;
  end
  复制代码
  ```

  1. 类似于多重if

  ```sql
  case 
  when 条件1 then 要显示的值1或语句1
  when 条件2 then 要显示的值2或语句2
  ...
  else 要显示的值n或语句n
  end
  ```

