

- [导读](#--)
- [基本界面](#----)
- [变量查看](#----)
- [计算表达式](#-----)
- [断点条件设置](#------)
- [线程切换](#----)
- [强制抛异常](#-----)
- [强制返回](#----)

## 导读

- 作为一名资深的老司机，IDEA调试可以说是家常便饭，如果不会debug，我都不信你读过源码，就别和我说原理了，直接pass掉。

## 基本界面

[![img](http://qa0a9jn2t.bkt.clouddn.com/aHR0cHM6Ly9pbWFnZXMyMDE3LmNuYmxvZ3MuY29tL2Jsb2cvODU2MTU0LzIwMTcwOS84NTYxNTQtMjAxNzA5MDUyMjE0MTgxNDctMTIwNTA0MzAyMC5wbmc.png)](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE3LmNuYmxvZ3MuY29tL2Jsb2cvODU2MTU0LzIwMTcwOS84NTYxNTQtMjAxNzA5MDUyMjE0MTgxNDctMTIwNTA0MzAyMC5wbmc)

① 以Debug模式启动服务，左边的一个按钮则是以Run模式启动。在开发中，我一般会直接启动Debug模式，方便随时调试代码。

② 断点：在左边行号栏单击左键，或者快捷键Ctrl+F8 打上/取消断点，断点行的颜色可自己去设置。

③ Debug窗口：访问请求到达第一个断点后，会自动激活Debug窗口。如果没有自动激活，可以去设置里设置。

④ 调试按钮：一共有8个按钮，调试的主要功能就对应着这几个按钮，鼠标悬停在按钮上可以查看对应的快捷键。

⑤ 服务按钮：可以在这里关闭/启动服务，设置断点等。

⑥ 方法调用栈：这里显示了该线程调试所经过的所有方法，勾选右上角的[Show All Frames]按钮，就不会显示其它类库的方法了，否则这里会有一大堆的方法。

⑦ Variables：在变量区可以查看当前断点之前的当前方法内的变量。

⑧ Watches：查看变量，可以将Variables区中的变量拖到Watches中查看 。

## 变量查看

- 在调试过程中往往需要观察变量的变化来判断业务逻辑，我们可以在以下的四个地方观察。

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032301.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032301.png)

① 最常用的变量的观察区域variables

② IDEA中最人性化的地方之一，会将变量的值阴影显示在变量的后面。

③ watch区域，眼镜的形状，一般不会展开。如下图：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032302.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032302.png)

点击’+’号可以新增需要观察的变量，点击’-‘号可以删除。

④ 鼠标悬停在变量上也会出现变量的值，点击展开即可查看。

## 计算表达式

- 在调试业务逻辑的时候一般总会遇到某个条件或者某个变量的计算值的还不知道的情况下就需要判断下一行代码，那么此处就需要用到计算表达式的功能。计算表达式有两种方法，如下：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032303.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032303.png)

① 选择需要计算的代码，鼠标右键—->Evaluate Expression—>Evaluate即可计算。

② 直接点击计算器形状控件即可弹出计算的窗口，将代码复制进去即可，注意复制进去的代码一定要符合逻辑，比如局部变量一定要是已经声明的。

## 断点条件设置

- 对于新手要看Spring源码的话，再遇到调试UserService的doGetBean的方法时可能要崩溃，因为doGetBean在容器启动的时候可能会被调用几十次，你把断点打在doGetBean方法体中能让你生不如死。
- 设置断点条件有两种方式：
  - ①直接在断点上右键，添加condition条件即可。
  - [![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032304.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032304.png)
  - ② view breakpoints(ctrl+shift+8)显示所有的断点，在condition中添加条件即可。
  - [![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032305.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032305.png)

- 异常断点：设置了异常断点后，比如空指针异常，在程序出现需要拦截的异常时会自动定位到指定的行。如下图：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032306.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032306.png)

① ctrl+shift+F8显示所有断点，点击+号添加`Java Exception Breakpoints`。

② debug运行，一旦有代码出现该异常，会自动定位到指定代码。

## 线程切换

- 通常我们在调试的时候，一个请求过来被拦截了，此时想要发起另外一个请求是无法重新发的，因为另外一个请求被阻塞了，只有当前线程执行完成之后才会走其他的线程。在IDEA中可以改变一下阻塞级别，有两种方法：
  - 断点上右键—>选择Thread—->Make Default，如下图：
  - [![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032307.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032307.png)
  - 显示所有断点(crtl+shift+F8)，选中某一个断点，选择Thread，Make Default即可。如下图：
  - [![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032308.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032308.png)
- 设置了阻塞级别，此时就可以在线程切换了，如下图：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032309.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032309.png)

## 强制抛异常

这是IDEA 2018年加入的新功能，可以直接在调试中抛出指定的异常。使用方法跟上面的弃栈帧类似，右击栈帧并选择**Throw Exception**，然后输入抛异常的代码，比如`throw new NullPointerException`，操作如下图：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032310.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032310.png)

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032311.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032311.png)

## 强制返回

- 这是IDEA2015版时增加的功能，类似上面的手动抛异常，只不过是返回一个指定值罢了。使用方法跟上面也都类似，右击栈帧并选择**Force Return**，然后输入要返回的值即可。如果是`void`的方法那就更简单了，连返回值都不用输。如下图：

[![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032312.png)](https://gitee.com/chenjiabing666/Blog-file/raw/master/IDEA/032312.png)