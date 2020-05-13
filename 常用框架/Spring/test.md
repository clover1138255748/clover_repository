

<!--ts-->

<!--te-->

熟悉Github的同学可能知道创建一个Repo，通常都会生成一个README.md。好的README能增加代码的可阅读性。另外通常也可以将README作为开发文档。而这个README本身是遵循Markdown语法的，但是Markdown本身并没有绝对标准，Github的渲染方式与一些常用博客渲染方式不相同，导致在使用时有些麻烦。这里推荐一个Github上的教程。

[GFM教程](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fguodongxiaren%2FREADME%2Fblob%2Fmaster%2FREADME.md)

[GFM教程博客地址](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fkaitiren%2Farticle%2Fdetails%2F38513715)

事实上大部分和普通Markdown还是类似的，但是目录的语法差别蛮大，刚好对于笔者而言，最近需要在Github上文档上建立目录来使用，但是又不想写GFM的语法。这个时候刚好搜索到了一些可以用的开源代码。这里简单介绍一个目前使用的方法。

# 1 Github+百度搜索结果

![img](https:////upload-images.jianshu.io/upload_images/3229757-ded8dc8a31cd7989.png?imageMogr2/auto-orient/strip|imageView2/2/w/1027/format/webp)

事实上解决方案还蛮多的（Github大法好）。

当时还在百度上搜索了下，找到了这个方案。

[ghtoc Github地址（pyhon）](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fsk1418%2Fghtoc)

[ghtoc博客](https://link.jianshu.com?t=https%3A%2F%2Fwww.v2ex.com%2Ft%2F151106)

# 2 解决方案：gh-md-toc

后面发现了gh-md-toc这个神器。

[gh-md-toc Github地址](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fekalinin%2Fgithub-markdown-toc)

但是这个东西在Mac和Linux很友好，windows似乎不那么友好。不过这里也给了windows的解决方案。

![img](https:////upload-images.jianshu.io/upload_images/3229757-31243336441b1b39.png?imageMogr2/auto-orient/strip|imageView2/2/w/903/format/webp)

就是github-markdown-toc.go。

[github-markdown-toc.go Github地址](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fekalinin%2Fgithub-markdown-toc.go)

如果你有GO语言（又是你）的编译环境，可以尝试自己编译，如果没有，可以直接下载编译好的二进制文件。

[二进制文件](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fekalinin%2Fgithub-markdown-toc.go%2Freleases)

下载下来之后，发现没有后缀名无法识别，实际上这是个exe文件，所以只需要暴力地在后面加上.exe就可以开始愉快使用了。

首先将README.md文档复制到gh-md-toc.exe的根目录下。

接着按住shift键同时右击。



作者：G小调的Qing歌
链接：https://www.jianshu.com/p/302abe331dcb
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
