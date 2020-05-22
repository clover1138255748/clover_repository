上一章中，我们创建了版本库runoob01,URL为svn://192.168.0.1/runoob01,svn用户user01有读写权限。 我们就可以通过这个URL在客户端对版本库进行检出操作。 svn checkout http://svn.server.com/svn/project_repo --username=user01 以上命令将产生如下结果：

```
    root@runoob:~/svn# svn checkout svn://192.168.0.1/runoob01 --username=user01
    A    runoob01/trunk
    A    runoob01/branches
    A    runoob01/tags
    Checked out revision 1.
```

检出成功后在当前目录下生成runoob01副本目录。查看检出的内容

```
    root@runoob:~/svn# ll runoob01/
    total 24
    drwxr-xr-x 6 root root 4096 Jul 21 19:19 ./
    drwxr-xr-x 3 root root 4096 Jul 21 19:10 ../
    drwxr-xr-x 2 root root 4096 Jul 21 19:19 branches/
    drwxr-xr-x 4 root root 4096 Jul 21 19:19 .svn/
    drwxr-xr-x 2 root root 4096 Jul 21 19:19 tags/
    drwxr-xr-x 2 root root 4096 Jul 21 19:19 trunk/
```

你想查看更多关于版本库的信息，执行 info 命令。