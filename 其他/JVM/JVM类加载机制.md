Java中的所有类，都需要由类加载器装载到JVM中才能运行。**类加载器本身也是一个类**，而它的工作就是把class文件从硬盘读取到内存中。在写程序的时候，我们几乎不需要关心类的加载，因为这些都是隐式装载的，除非我们有特殊的用法，像是反射，就需要显式的加载所需要的类。

Java类的加载是动态的，它并不会一次性将所有类全部加载后再运行，而是保证程序运行的基础类(像是基类)完全加载到jvm中，至于其他类，则在需要的时候才加载。这当然就是为了节省内存开销。

Java的类加载器有三个，对应Java的三种类:

![image-20200722154928013](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200722154928013.png)

三个加载器各自完成自己的工作，但它们是如何协调工作呢？哪一个类该由哪个类加载器完成呢？为了解决这个问题，Java采用了委托模型机制。

委托模型机制的工作原理很简单：当类加载器需要加载类的时候，先请示其Parent(即上一层加载器)在其搜索路径载入，如果找不到，才在自己的搜索路径搜索该类。这样的顺序其实就是加载器层次上自顶而下的搜索，因为加载器必须保证基础类的加载。之所以是这种机制，还有一个安全上的考虑：如果某人将一个恶意的基础类加载到jvm，委托模型机制会搜索其父类加载器，显然是不可能找到的，自然就不会将该类加载进来。

我们可以通过这样的代码来获取类加载器:

![image-20200722155018996](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200722155018996.png)

注意一个很重要的问题，就是Java在逻辑上并不存在BootstrapKLoader的实体！因为它是用C++编写的，所以打印其内容将会得到null。
前面是对类加载器的简单介绍，它的原理机制非常简单，就是下面几个步骤:

1. 装载：查找和导入Class文件；
2. 链接：执行校验、准备和解析步骤，其中解析步骤是可以选择的：

![image-20200722155112775](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200722155112775.png)

3. 初始化：对类的静态变量、静态代码块执行初始化工作。

![image-20200722155138145](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200722155138145.png)

## 什么是类加载器?

类加载器是一个用来加载类文件的类。Java源代码通过javac编译器编译成类文件。然后JVM来执行类文件中的字节码来执行程序。类加载器负责加载文件系统、网络或其他来源的类文件。

## 类加载器有哪些?

### Bootstrap类加载器

Bootstrap类加载器负责加载rt.jar中的JDK类文件，它是所有类加载器的父加载器。Bootstrap类加载器没有任何父类加载器，如果你调用String.class.getClassLoader()，会返回null，任何基于此的代码会抛出NullPointerException异常。Bootstrap加载器被称为初始类加载器。

### Extension类加载器

而Extension将加载类的请求先委托给它的父加载器，也就是Bootstrap，如果没有成功加载的话，再从jre/lib/ext目录下或者java.ext.dirs系统属性定义的目录下加载类。Extension加载器由sun.misc.Launcher$ExtClassLoader实现。

### Application类加载器

第三种默认的加载器就是Application类加载器了。它负责从classpath环境变量中加载某些应用相关的类，classpath环境变量通常由-classpath或-cp命令行选项来定义，或者是JAR中的Manifest的classpath属性。Application类加载器是Extension类加载器的子加载器。通过sun.misc.Launcher$AppClassLoader实现。

## 类加载器双亲委派模型机制

什么是双亲委派模型(Parent-Delegation Model)？为什么使用双亲委派模型？

JVM中加载类机制采用的是双亲委派模型，顾名思义，在该模型中，子类加载器收到的加载请求，不会先去处理，而是先把请求委派给父类加载器处理，当父类加载器处理不了时再返回给子类加载器加载

为什么使用双亲委派模型？

因为安全。使用双亲委派模型来组织类加载器间的关系，能够使类的加载也具有层次关系，这样能够保证核心基础的Java类会被根加载器加载，而不会去加载用户自定义的和基础类库相同名字的类，从而保证系统的有序、安全。