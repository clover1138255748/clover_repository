通过svn命令可以根据时间或修订号去除过去的版本，或者某一版本所做的具体的修改。以下四个命令可以用来查看svn 的历史：

- **svn log:** 用来展示svn 的版本作者、日期、路径等等。
- **svn diff:** 用来显示特定修改的行级详细信息。
- **svn cat:** 取得在特定版本的某文件显示在当前屏幕。
- **svn list:** 显示一个目录或某一版本存在的文件。

------

## 1、svn log

可以显示所有的信息，如果只希望查看特定的某两个版本之间的信息，可以使用：

```
    root@runoob:~/svn/runoob01/trunk# svn log -r 6:8
    ------------------------------------------------------------------------
    r6 | user02 | 2016-11-07 02:01:26 +0800 (Mon, 07 Nov 2016) | 1 line

    change HelloWorld.html first.
    ------------------------------------------------------------------------
    r7 | user01 | 2016-11-07 02:23:26 +0800 (Mon, 07 Nov 2016) | 1 line

    change HelloWorld.html second
    ------------------------------------------------------------------------
    r8 | user01 | 2016-11-07 02:53:13 +0800 (Mon, 07 Nov 2016) | 1 line

    SVN readme.
    ------------------------------------------------------------------------
```

如果只想查看某一个文件的版本修改信息，可以使用 **svn log** 文件路径。

```
    root@runoob:~/svn/runoob01# svn log trunk/HelloWorld.html 
    ------------------------------------------------------------------------
    r7 | user01 | 2016-11-07 02:23:26 +0800 (Mon, 07 Nov 2016) | 1 line

    change HelloWorld.html second
    ------------------------------------------------------------------------
    r6 | user02 | 2016-11-07 02:01:26 +0800 (Mon, 07 Nov 2016) | 1 line

    change HelloWorld.html first.
    ------------------------------------------------------------------------
    r5 | user01 | 2016-11-07 01:50:03 +0800 (Mon, 07 Nov 2016) | 1 line


    ------------------------------------------------------------------------
    r4 | user01 | 2016-11-07 01:45:43 +0800 (Mon, 07 Nov 2016) | 1 line

    Add function to accept input and to display array contents
    ------------------------------------------------------------------------
    r3 | user01 | 2016-11-07 01:42:35 +0800 (Mon, 07 Nov 2016) | 1 line


    ------------------------------------------------------------------------
    r2 | user01 | 2016-08-23 17:29:02 +0800 (Tue, 23 Aug 2016) | 1 line

    first file
    ------------------------------------------------------------------------
```

如果希望得到目录的信息要加 **-v**。 如果希望显示限定N条记录的目录信息，使用 **svn log -l N -v**。

```
    root@runoob:~/svn/runoob01/trunk# svn log -l 5 -v 
    ------------------------------------------------------------------------
    r6 | user02 | 2016-11-07 02:01:26 +0800 (Mon, 07 Nov 2016) | 1 line
    Changed paths:
       M /trunk/HelloWorld.html

    change HelloWorld.html first.
    ------------------------------------------------------------------------
    r5 | user01 | 2016-11-07 01:50:03 +0800 (Mon, 07 Nov 2016) | 1 line
    Changed paths:
       M /trunk/HelloWorld.html


    ------------------------------------------------------------------------
    r4 | user01 | 2016-11-07 01:45:43 +0800 (Mon, 07 Nov 2016) | 1 line
    Changed paths:
       M /trunk/HelloWorld.html

    Add function to accept input and to display array contents
    ------------------------------------------------------------------------
    r3 | user01 | 2016-11-07 01:42:35 +0800 (Mon, 07 Nov 2016) | 1 line
    Changed paths:
       A /trunk/HelloWorld.html (from /trunk/helloworld.html:2)
       D /trunk/helloworld.html


    ------------------------------------------------------------------------
    r2 | user01 | 2016-08-23 17:29:02 +0800 (Tue, 23 Aug 2016) | 1 line
    Changed paths:
       A /trunk/helloworld.html

    first file
    ------------------------------------------------------------------------
```

------

## 2、svn diff

用来检查历史修改的详情。

- 检查本地修改
- 比较工作拷贝与版本库
- 比较版本库与版本库

**（1）、如果用 svn diff，不带任何参数，它将会比较你的工作文件与缓存在 .svn 的"原始"拷贝。**

```
    root@runoob:~/svn/runoob01/trunk# svn diff
    Index: rules.txt
    ===================================================================
    --- rules.txt (revision 3)
    +++ rules.txt (working copy)
    @@ -1,4 +1,5 @@
    Be kind to others
    Freedom = Responsibility
    Everything in moderation
    -Chew with your mouth open
```

**（2）、比较工作拷贝和版本库** 比较你的工作拷贝和版本库中版本号为 3 的文件 rule.txt。

```
    svn diff -r 3 rule.txt
```

**（3）、比较版本库与版本库** 通过 -r(revision) 传递两个通过冒号分开的版本号，这两个版本会进行比较。 比较 svn 工作版本中版本号2和3的这个文件的变化。

```
    svn diff -r 2:3 rule.txt
```

------

## 3、svn cat

如果只是希望检查一个过去版本，不希望查看他们的区别，可使用svn cat

```
    svn cat -r 版本号 rule.txt
```

这个命令会显示在该版本号下的该文件内容

------

## 4、svn list

s**vn list** 可以在不下载文件到本地目录的情况下来察看目录中的文件：

```
    $ svn list http://192.168.0.1/runoob01
    README
    branches/
    clients/
    tags/
```