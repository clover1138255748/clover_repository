[TOC]

## 产品简介

- FineReport 是帆软自主研发的企业级 web 报表工具，经过多年的打磨，已经成长为中国报表软件领导品牌。
- FineReport 以其零编码的理念，易学易用，功能强大，简单拖拽操作便可制作中国式复杂报表。

![image-20200713104239086](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200713104239086.png)

### 设计器安装

链接: https://pan.baidu.com/s/1IVl656HDw2o0WFz34oxKhw  密码: sbmi

下载后里面有文档

### 设计器界面概览

![image-20200713104708457](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200713104708457.png)



## FineReport文件系统

### 工作目录reportlets

C:\FineReport_9.0\WebReport\WEB-INF\reportlets

这个文件夹主要对应你的开发环境

![image-20200713105202659](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200713105202659.png)

就是存放cpt文件的地方

### lib目录

![image-20200713105337171](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200713105337171.png)

这里主要是放jar包的地方,因为这个报表本身就是一个单独的tomcat  

这里差不多是我工作用的所以jar  如果你程序运行不起来就看看是不是缺失了jar

### class目录

C:\FineReport_9.0\WebReport\WEB-INF\classes\com\fr\data

这个目录对应的是你需要java代码开发的时候  你写了一个java程序  

然后会生成一个class 

你把这个class放这里 然后报表里面引用这个class



## FineReport开发一张简单报表

首先我们先看看效果

![image-20200720172350449](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720172350449.png)



### 第一步 新建数据连接

新建数据连接的目的是让 FineReport 设计器连接数据库，这样报表就可以在数据库中读取、写入或修改数据。

数据连接的方式有两种，分别是连接内置数据库和连接外置数据库。制作这张报表连接的是 FineReport 内置的 SQLite 类型的数据库，有关外置数据库的连接可参见 [JDBC连接数据库](https://help.finereport.com/doc-view-101.html)。

1）打开设计器，菜单栏选择服务器>定义数据连接。

![Snag_4d21c01.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1566886402416196.png)

2）弹出「定义数据连接」对话框，设计器已经默认连接了一个名为 FRDemo 的内置数据库，点击测试链接，弹出「连接成功」提示框，表示数据库 FRDemo 成功与设计器建立连接。接下来就可以从这个数据库中取数用于报表的设计。

![Snag_453480c.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1566878091486915.png)

### 第二步 新建报表类型

菜单栏选择文件>新建普通报表或者点击新建普通报表按钮![1.jpg](https://help.finereport.com/uploads/20190827/1566890499165160.jpg)，新建一张空白的普通报表。

![Snag_4d4ae74.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1566886560969288.png)

### 第三步 新建数据集

数据集通过 SQL 查询语句从已经建立连接的数据库中取数，将数据以二维表的形式保存并显示在数据集管理面板处。简单而言数据集是报表设计时的直接数据来源。

数据集按照作用范围分为两种：服务器数据集 和 模板数据集，它们之间的区别请参见：[数据集](https://help.finereport.com/doc-view-106.html)。

我们制作的这张普通报表将新建两个模板数据集 ds1 和 ds2。

1）数据集管理面板选择模板数据集，点击上方的![2.jpg](https://help.finereport.com/uploads/20190827/1566890482341691.jpg)，在弹出的模板数据集类型选择框中点击数据库查询。

![Snag_4d91e80.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1566886851320253.png)

2）在弹出的数据库查询对话框中，写入数据查询语句，新建数据集注册数和累计，查询写好sql查询出表中的所有数据。

![Snag_a7270eb.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1566980824120780.png)

3）新建好数据集之后，可在数据集管理面板查看取出的数据。

![image-20200720172814929](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720172814929.png)

### 第四步 表格设计

就和excel一样差不多

### 第五步 数据展示 

将数据拖入到框内展示

![image-20200720173031239](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720173031239.png)

这样就完成了一张报表的开发

具体细节参考  https://help.finereport.com/doc-view-72.html

## FineReport开发一张复杂报表

### 利用代码开发

FineReport 报表的数据来源可以是数据库数据或是文本数据，并且还可以是其它任何类型的数据，因为 FineReport 是通过 AbstractTableData 抽象类来读取数据源的，而上述所有的数据来源都继承实现其抽象方法，因此用户只要实现了 AbstractTableData 抽象类，也就可以用自定义类型的数据源了（程序数据集），FineReport 报表引擎就能够读取定义的数据源作为报表数据源使用

### 原理



AbstractTableData 抽象类主要有5个方法，如下：

**//获取 AbstractTableData 的总列数**

```
public int getColumnCount();
```

**//获取 AbstractTableData 中第 columnIndex 列的列名**

```
public String getColumnName(int columnIndex);
```

**//判断是否存在第 rowIndex 行，这主要是用于处理超大数据时，完全遍历所有数据获取总行数相当困难，用这个方法来判断第 rowIndex 行是否存在，存在则可读取**

```
public boolean hasRow(int rowIndex);
```

**//获取 AbstractTableData 的总行数**

```
public int getRowCount();
```

**//获取 AbstractTableData 中第 columnIndex 列，第 rowIndex 行的数据**

```
public Object getValueAt(int rowIndex, int columnIndex);
```

在某些应用场景中，需要在程序中对数据进行处理后再作为报表的数据源使用。

### 第一步 定义程序数据源

定义一个类，继承 AbstractTableData，并实现里面的方法，

![image-20200720174155056](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720174155056.png)

#### 定义参数

![image-20200720174245599](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720174245599.png)

#### 发送http请求

![image-20200720174328098](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720174328098.png)

#### 定义返回参数

![image-20200720174401243](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720174401243.png)

这样要和这里对应

![image-20200720174422284](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200720174422284.png)



### 编译 class 文件

将 ArrayTableDataDemo.java 编译生成 ArrayTableDataDemo.class 类。

将生成的类文件拷贝到报表工程 %FR_HOME%\webapps\webroot\WEB-INF\classes 目录下。由于该类是在 com.fr.data 包中的，因此最终应该将该 ArrayTableData.class 放在 %FR_HOME%\webapps\webroot\WEB-INF\classes\com\fr\data 下面。此时该程序数据源便定义好了。

### 配置程序数据源

点击模板数据集下面的加号，选择程序数据集，然后在弹出的程序数据集对话框中，选择对应的 class 文件，如下图：

![222](https://gitee.com/cdx_dayshow/picBed/raw/master/img/1552369633dr5hL4d2.png)

插入图表完事