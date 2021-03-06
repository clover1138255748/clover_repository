当我们想放弃对文件的修改，可以使用 **SVN revert** 命令。 svn revert 操作将撤销任何文件或目录里的局部更改。 我们对文件 readme 进行修改,查看文件状态。

```
    root@runoob:~/svn/runoob01/trunk# svn status
    M       readme
```

这时我们发现修改错误，要撤销修改，通过 svn revert 文件 readme 回归到未修改状态。

```
    root@runoob:~/svn/runoob01/trunk# svn revert readme 
    Reverted 'readme'
```

再查看状态。

```
    root@runoob:~/svn/runoob01/trunk# svn status 
    root@runoob:~/svn/runoob01/trunk# 
```

进行 revert 操作之后，readme 文件恢复了原始的状态。 revert 操作不单单可以使单个文件恢复原状， 而且可以使整个目录恢复原状。恢复目录用 -R 命令，如下。

```
    svn revert -R trunk
```

但是，假如我们想恢复一个已经提交的版本怎么办。 为了消除一个旧版本，我们必须撤销旧版本里的所有更改然后提交一个新版本。这种操作叫做 reverse merge。 首先，找到仓库的当前版本，现在是版本 22，我们要撤销回之前的版本，比如版本 21。

```
    svn merge -r 22:21 readme 
```