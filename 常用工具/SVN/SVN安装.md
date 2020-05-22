## 在windows下安装 SVN

1、准备svn的安装文件 下载地址：https://sourceforge.net/projects/win32svn/[![201909251002_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201909251002_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/cainaiaojiaocheng/201909251002_1.png)2、下载完成后，在相应的盘符中会有一个Setup-Subversion-1.8.16.msi的文件，目前最新的版本是1.8.16， 这里就使用这个版本。然后双击安装文件进行安装。我们指定安装在D:\Program Files (x86)\Subversion目录里。[![201909251002_2.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201909251002_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/cainaiaojiaocheng/201909251002_2.png)3、查看目录结构[![201909251002_3.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201909251002_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/cainaiaojiaocheng/201909251002_3.png)把svn安装目录里的bin目录添加到path路径中，在命令行窗口中输入 svnserve --help ,查看安装正常与否。[![201909251002_4.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201909251002_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/cainaiaojiaocheng/201909251002_4.png)至此，windows下的SVN安装完成

------

## 在CentOS下安装 SVN

大多数 GNU/Linux 发行版系统自带了Subversion ，所以它很有可能已经安装在你的系统上了。可以使用下面命令检查是否安装了。

```
    svn --version
```

如果 Subversion 客户端没有安装，命令将报告svn命令找不到的错误。

```
    [runoob@centos6 ~]$ svn --version
    bash: svn: command not found
```

我们可以使用 yum install subversion 命令进行安装。

```
    [runoob@centos6 root]$ su -
    密码：
    [root@centos6 ~]# yum install subversion
    已加载插件：fastestmirror, security
    设置安装进程
    Loading mirror speeds from cached hostfile
     * base: mirrors.aliyun.com
     * epel: mirrors.neusoft.edu.cn
     * extras: mirrors.zju.edu.cn
     * updates: mirrors.aliyun.com
    解决依赖关系
    --> 执行事务检查
    ...
```

安装成功之后，执行 svn --version 命令。

```
    [root@centos6 ~]# svn --version
    svn，版本 1.6.11 (r934486)
       编译于 Aug 17 2015，08:37:43
```

至此，centos下的SVN安装完成。

------

## 在Ubuntu下安装 SVN

如果 Subversion 客户端没有安装，命令将报告svn命令找不到的错误。

```
    root@runoob:~# svn --version
    The program 'svn' is currently not installed. You can install it by typing:
    apt-get install subversion
```

我们可以使用 apt-get 命令进行安装

```
    root@runoob:~# apt-get install subversion
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
      augeas-lenses hiera libaugeas0 libxslt1.1 ruby-augeas ruby-deep-merge ruby-json ruby-nokogiri ruby-rgen ruby-safe-yaml ruby-selinux ruby-shadow
    Use 'apt-get autoremove' to remove them.
    The following extra packages will be installed:
      libserf-1-1 libsvn1
    ...
```

安装成功之后，执行 svn --version 命令。

```
    root@runoob:~# svn --version
    svn, version 1.8.13 (r1667537)
       compiled Sep  8 2015, 14:59:01 on x86_64-pc-linux-gnu
```

至此，Ubuntu下的SVN安装完成。

## 在MAC下安装

还安装个p啊 自带的