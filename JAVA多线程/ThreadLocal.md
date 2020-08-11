### **第一节：线程上下文**

进程是资源的分配单位，线程是运行调度单位。也就是说，任何运行的程序，必定归属于某个线程。不管是main线程也好，还是其他的线程也罢。这一点要清楚。

程序跑起来，就会产生一个线程。在这个线程里面会有一个context上下文，我们可以往context里面存放东西，随后在线程管辖范围内都可以获取到。伪代码示例如下所示：

```
//此处是伪代码
Thread t = Thread.currentThread();

ThreadContext context = t.getContext();

//存数据
context.set(1000);

//取数据
context.get();
```

### **第二节：ThreadLocal的原理介绍**

线程本来就属于重型对象，现在还带个上下文，岂不是重上加重，胖的有点跑不动了。所以，JDK的作者耍了一个小聪明：用的时候再创建这个context上下文，不用则留个占位就行了。

如何实现这种“延迟创建线程上下文context”的目的呢？jdk作者采用的方案是：用threadlocal来负责创建上下文。

```
Thread t = Thread.currentThread();

ThreadContext context = t.getContext();

if ( context == null )
{
    context  = createContext();
}
```

### **第三节：ThreadLocal的用法介绍**

ThreadLocal是线程上下文context的代理对象，context的目的是存放数据，自然ThreadLocal也是用来存放数据，所以主要用法就是set和get操作。

ThreadLocal在set存数据到线程上下文context的时候，把自己[this]也放进去了。也就是这样的：

```
class ThreadLocal{

	public void set()
	{
		ThreadContext context = t.getContext();

		if ( context = null )
		{
			context  = createContext();
		}

		context.set(this,1000);
	}

}
```

ThreadLocal为什么要把自己放进去呢？因为线程的上下文只有一个，但是ThreadLocal有多个，是谁往线程上下文里面放东西，得有个登记，登记的形式是：key-value的格式，否则，ThreadLocalA和ThreadLocalB都从context里面掏东西，都乱套了，掏出来的东西不是自己之前放进去的，岂不是傻眼了？

### **第四节：ThreadLocal的内存泄漏**

什么是内存泄漏呢？简单的说，就是东西放在内存里面，但你忘记它放哪里了，它占着一块内存，但是不能回收。当这样的东西越来越多，内存就吃紧，最终导致服务器宕机。

再讲一个小故事，阐述一下内存泄漏。在抗日时期，有两名地下党A和B，A是上线，B是下线，B不能直接联系党中央的，他需要通过A来帮忙传话。一旦A发生意外，党中央就找不到B了，B一直存在，但是茫茫人海，党中央是无法启用B做战斗任务的安排，这种情况类似内存泄漏。

ThreadLocal的内存分配是这样的：

```
//新建一个ThreadLocal变量num，此时是一个强引用
ThreadLocal<Integer> num = new ThreadLocal<Integer>();

//set数据之后，再增加一个弱引用，此时计数器为2
num.set(10)

//强引用不再被使用，系统回收强引用，计数器减1，此时只存在一个弱引用
...

//弱引用不稳定，很容易被回收，一旦被回收，其登记的key-value形式的数据，此时就变成了null-value
//key都消失了，value就找不到了，造成内存泄漏
...
```

### **第五节：既然ThreadLocal会出现内存泄漏，那为什么用弱引用，不用强引用呢？**

问题的分析过程如下：

ThreadLocal是由两个维度组成：线程和变量。从变量的角度理解，往往才能更深刻的理解这个问题。

既然ThreadLocal是变量，那么变量用完之后，应该被JVM自动清理，这是Java的特点和优势。代码如下所示：

```
public class ThreadLocalDemo
{
	public static void main(String[] args)
	{
		ThreadLocal<Integer> count = new ThreadLocal<Integer>();
		//使用count
		count.set(10);
		// count不再被使用，可以进行内存回收
		System.out.println("");
	}
}
```

如上代码所示，count本身是程序员能感知到的东西，这类东西不用了就应该清理的，但是它现在被用在了程序员看不到的地方，即：count和10被存入了ThreadLocalMap里面。

我想，此时JDK的作者也是左右为难，从眼见为实的角度来说，变量不用了就应该进行回收，实现内存的自动回收，这是Java给人最大的特点。

但是，此时不能回收count，因为它与10绑定到了一块，而且只能通过count才能读写10。最后JDK作者耍了一个小聪明，用弱引用包装了count，没有干脆利索的进行内存回收，而是拖拖拉拉的进行回收，反正，最后实现了变量不用就回收的基本原则，与Java的传统思想一脉相承。

ThreadLocal本质就是变量，一切思考要从变量的角度去想问题，而不是从线程的角度考虑问题。

通常情况下，人们对ThreadLocal的理解思路是这样的：首先把它放在线程这个大的背景下去琢磨，然后认为它是解决多线程的变量冲突问题。

但是，按照上述这样的思路去学习，其实效果不好。由于"线程"这个东西是抽象的东西，对于所有的人来说，"线程"就是一只拦路虎，让每一个接触ThreadLocal的人都产生了内心的抵触，产生了虚无缥缈的无助感，所以我们对ThreadLocal的理解没有深度，没有灵性，很教条，很空洞。

### **第六节：为什么ThreadLocal令我们难以理解？**

ThreadLocal是由两个维度组成：线程和变量。

通常情况下，人们对ThreadLocal的理解思路是这样的：首先把它放在线程这个大的背景下去琢磨，然后认为它是解决多线程的变量冲突问题。这种思维方式的流动方向是这样的：线程->变量。

但是按照这样的思路去学习，其实效果不好。由于"线程"这个东西是抽象的东西，对于所有的人来说，"线程"就是一只拦路虎，让每一个接触ThreadLocal的人都产生了内心的抵触，产生了虚无缥缈的无助感，所以我们对ThreadLocal的理解没有深度，没有灵性，很教条，很空洞。

何不抛开线程而从变量的角度来认识和掌握 ThreadLocal呢？毕竟ThreadLocal是由两个维度组成：线程和变量。所以我倡导的思维方式是这样的：变量->线程。

##### 1、ThreadLocal的变量属性

抛开线程的属性，ThreadLocal就是一种类型的变量而已。对于普通的变量而言，我们很熟悉，其操作过程无非就是这样的：

```
int a; //定义变量，开辟一块内存

a = 10; //给变量赋值
```

如上代码所示，整数字面量10存储到了变量a里面。a表示一块内存。当a不再被使用的时候，会被Java虚拟机所回收。



对于同样是变量的ThreadLocal而言，它的用法是这样的：

```
//声明一个ThreadLocal变量b，此时开辟了一块内存，里面存放的b对象
ThreadLocal<Integer> b = new ThreadLocal<Integer>();

//注意，这里不是赋值，赋值是=操作符
b.set(10)
```

在上面这个代码里面，10是放在了b里面吗？这一点令人非常的困惑！一定要记住牢记下面两点：

（1）10不是放在了b里面，10和b是两个独立存放的东西，不是包含关系。

（2）10和b是两个独立存放的变量，如果其中的一个被清理，那么另外一个不受影响的。

既然10和b是独立存放的，那么它们之间到底有什么关系呢？其实，这种关系，可以通过一个小故事来阐述清楚：

 **在抗日时期，有两名地下党A和B，A是上线，B是下线，B不能直接联系党中央的，他需要通过A来帮忙传话。一旦A发生意外，党中央就找不到B了，B一直存在，但是茫茫人海，党中央是无法启用B做战斗任务的安排，这种情况类似内存泄漏。**

##### 2、数据存放在哪里？

如上代码所示，10和b存放在哪里呢？线程活着，都有一个map，类似于context，每个线程都有一个map，随时随地可以存放东西。既然是map，肯定是用key-value的形式存在，key就是b，value就是10。

##### 3、薄命郎：弱引用

10和b两个的独立存放的东西，只不过我们不能直接访问到10，必须通过b来传话，原因很简单，在map里面，value要通过key来访问。

不过，此时b被包装成了弱引用，也就是说它被打了一个标签，这样它很容易被gc。一旦b被清理了，10就找不到了，从而造成了内存泄漏。

总之，b是个薄命郎，用他来做上线，他很容易挂掉的。上线挂了，下线自然就联系不上了。

##### 4、小结

（1）有两个变量a和b，如果后面不再参与计算，则会被自动回收；

（2）有两个变量a和b，存放在map里面，a是key，b是value。如果map一直存在，a和b因为被map关联，则a和b就一直不能被回收；

备注：线程活着，都有map，类似于context，每个线程都有一个map，好比一个context一样，随时随地可以存放东西

（3）有两个变量a和b，存放在map里面，a是key（但是a被弱引用包装了一下），b是value。如果map一直存在，a和b因为被map关联，则b就一直不能被回收，但是a可以被回收。一旦a回收了，那么无法通过a找到b了，这就是b出现内存泄漏。



### **第七节：一针见血理解ThreadLocal类**

ThreadLocal类具有两个维度：线程维度和变量维度。扔掉线程维度，保留并放大变量维度，虽然思想片面，但是给人的印象却是极深，才能用之出神入化。 如果丁是丁，卯是卯，分析的很全面，也不过是纸上谈兵，因为用的时候拼的是感觉。

ThreadLocal类是修饰变量的，重点是在控制变量的作用域，初衷可不是为了解决线程并发和线程冲突的，而是为了让变量的种类变的更多更丰富，方便人们使用罢了。很多开发语言在语言级别都提供这种作用域的变量类型。

根据变量的作用域，可以将变量分为全局变量，局部变量。简单的说，类里面定义的变量是全局变量，函数里面定义的变量是局部变量。

还有一种作用域是线程作用域，线程一般是跨越几个函数的。为了在几个函数之间共用一个变量，所以才出现：线程变量，这种变量在Java中就是ThreadLocal变量。

全局变量，范围很大；局部变量，范围很小。无论是大还是小，其实都是定死的。而线程变量，调用几个函数，则决定了它的作用域有多大。

ThreadLocal是跨函数的，虽然全局变量也是跨函数的，但是跨所有的函数，而且不是动态的。

ThreadLocal是跨函数的，但是跨哪些函数呢，由线程来定，更灵活。

```
class TreadLocalDemo
{
 int m = 0; //全局变量
 ThreadLocal<Integer> iThreadLocal = new ThreadLocal<Integer>();//线程变量
 void main()
 {
  int n = 0;//局部变量
 }
 void entry1()
 {
  int temp = iThreadLocal.get();
 }
 void entry2()
 {
  int temp = iThreadLocal.get();
 }
 void entry3()
 {
  int temp = iThreadLocal.get();
 }
}
```

假设有三个线程，则对应三种线程变量的三个不同的作用域： thread1: entry1-> entry2
thread2: entry2-> entry3
thread3: entry1-> entry2-> entry3

如上，线程变量的作用域更灵活吧。一个线程一个变量，而且线程跨越多少个函数，则这个变量也跨越多少个函数。

总之，ThreadLocal类是修饰变量的，是在控制它的作用域，是为了增加变量的种类而已，这才是ThreadLocal类诞生的初衷，它的初衷可不是解决线程冲突的。

### **第八节：ThreadLocal的应用场景：将类改造成上下文类**

类是数据的封装，是个容器。下面的例子是一个记录错误信息的类。初始的编码手法平淡无奇，让人读完之后，跟喝白开水一样，经过改造变得十分有内涵，有厚度，更新一件艺术品。

```
import java.util.ArrayList;
import java.util.List;

public class Error
{
	private List<String> messages = new ArrayList<String>();

	public Error()
	{

	}

	public Error message(String message)
	{
		this.messages.add(message);
		return this;
	}

	public Error reset()
	{
		messages.clear();
		return this;
	}

	@Override
	public String toString()
	{
		StringBuilder description = new StringBuilder();

		for (String msg : messages)
		{
			description.append("### ");
			description.append(msg);
			description.append("\n");
		}

		return description.toString();
	}

	public static void main(String[] args)
	{
		//新建一个Error类，将它分别用在三个线程里面：main，task1，task2
		// 这种编码用法非常平淡，没有特色，大家都这么用，最后Error被三个线程乱七八糟的塞进了各种东西
		final Error error = new Error();

		error.message("Main Thread Message");
		System.out.println(error);

		Runnable task1 = () -> {
			error.message("Task1 Thread Message");
			System.out.println(error);

		};

		Runnable task2 = () -> {
			error.message("Task2 Thread Message");
			System.out.println(error);

		};

		new Thread(task1).start();

		new Thread(task2).start();

	}
}
	
```

下面是运行结果，大家可以看出来，输出的结果已经有点乱了。

```
//main线程打出自己的错误信息
### Main Thread Message

//task1线程不仅打出自己的错误信息，把main线程的错误信息也打出
### Main Thread Message
### Task1 Thread Message

//task2线程更不着调，不仅打出自己的错误信息，把main线程和task1线程的错误信息都打出
### Main Thread Message
### Task1 Thread Message
### Task2 Thread Message
```

将上面的类改造一下，从Error变成了ErrorContext，多出一个Context，则意境发生了明显的变化。关于Context的写作手法，在《趣谈shell》节选二：精灵小黑，分身有术，已经有详细的介绍。在此不再赘述：http://ads.shelltalk.cn/。有点抱歉：《趣谈shell》因为运营成本太大，已经停售了，不再对外发售，仅供徒弟内部使用。

**注意：如果因为多线程问题，导致运行结果与上述不符，可以酌情在每个线程内部增加sleep方法，避免其跑的太快，要让它等等别人。**


在上述程序中，启动了三个线程，分别是：main线程，Task1线程，Task2线程。这三个线程是平等的，一旦它们开跑，谁先跑完，是由自己决定的，别人无权干涉。

虽然Task1线程和Task2线程是从main线程里面分叉出来的，但是一旦开跑，它们同时会跟main线程竞争CPU时间片，没有任何感恩main线程的孕育之心。

```
import java.util.ArrayList;
import java.util.List;

public class ErrorContext
{
	private List<String> messages = new ArrayList<String>();

	private static final ThreadLocal<ErrorContext> LOCAL = new ThreadLocal<ErrorContext>();

	private ErrorContext()
	{

	}

	public static ErrorContext getInstance()
	{
		ErrorContext context = LOCAL.get();
		if (context == null)
		{

			context = new ErrorContext();
			LOCAL.set(context);
		}
		return context;
	}

	public ErrorContext message(String message)
	{
		this.messages.add(message);
		return this;
	}

	public ErrorContext reset()
	{

		messages.clear();
		LOCAL.remove();
		return this;
	}

	@Override
	public String toString()
	{
		StringBuilder description = new StringBuilder();

		for (String msg : messages)
		{
			description.append("### ");
			description.append(msg);
			description.append("\n");
		}

		return description.toString();
	}

	public static void main(String[] args)
	{

		ErrorContext cxtMain = ErrorContext.getInstance();

		cxtMain.message("Main Thread Message");
		System.out.println(cxtMain);
		cxtMain.reset();

		Runnable task1 = () -> {

			ErrorContext cxtTask1 = ErrorContext.getInstance();
			cxtTask1.message("Task1 Thread Message");
			System.out.println(cxtTask1);
			cxtTask1.reset();
		};

		Runnable task2 = () -> {
			ErrorContext cxtTask2 = ErrorContext.getInstance();
			cxtTask2.message("Task2 Thread Message");
			System.out.println(cxtTask2);
			cxtTask2.reset();
		};

		new Thread(task1).start();

		new Thread(task2).start();

	}
}

```

```
### Main Thread Message

### Task1 Thread Message

### Task2 Thread Message
```

### **第九节：Java中的四种引用类型（强、软、弱、虚）**

从Java 1.2开始，JVM开发团队发现，单一的强引用类型，无法很好的管理对象在JVM里面的生命周期，垃圾回收策略过于简单，无法适用绝大多数场景。为了更好的管理对象的内存，更好的进行垃圾回收，JVM团队扩展了引用类型，从最早的强引用类型增加到强、软、弱、虚四个引用类型。

##### 强引用（Strong Reference）

Strong Rerence这个类并不存在，默认的对象都是强引用类型，因为有后来的新引用所衬托，所以才起了个名字叫"强引用"。

强引用使用示例如下所示：

```
String web = "xxx";
```

如果JVM垃圾回收器 GC 可达性分析结果为可达，表示引用类型仍然被引用着，这类对象始终不会被垃圾回收器回收，即使JVM发生OOM也不会回收。而如果 GC 的可达性分析结果为不可达，那么在GC时会被回收。

##### 软引用（Soft Reference）

软引用是一种比强引用生命周期稍弱的一种引用类型。在JVM内存充足的情况下，软引用并不会被垃圾回收器回收，只有在JVM内存不足的情况下，才会被垃圾回收器回收。所以软引用一般用来实现一些内存敏感的缓存，只要内存空间足够，对象就会保持不被回收掉。

软引用使用示例如下所示：

```
SoftReference<String> softReference = new SoftReference<String>(new String("xxx"));
String web = softReference.get();

```

##### 弱引用（Weak Reference）

弱引用是一种比软引用生命周期更短的引用。它的生命周期很短，不论当前内存是否充足，都只能存活到下一次垃圾收集之前。

```
WeakReference<String> weakReference = new WeakReference<String>(new String("xxx"));

System.gc();

if(weakReference.get() == null)
{
    System.out.println("weakReference已经被GC回收");
}
```

输出结果：

weakReference已经被GC回收

##### 虚引用（PhantomReference）

虚引用与前面的几种都不一样，这种引用类型不会影响对象的生命周期，所持有的引用就跟没持有一样，随时都能被GC回收。

需要注意的是，在使用虚引用时，必须和引用队列关联使用。在对象的垃圾回收过程中，如果GC发现一个对象还存在虚引用，则会把这个虚引用加入到与之关联的引用队列中。

程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。

如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象内存被回收之前采取必要的行动防止被回收。虚引用主要用来跟踪对象被垃圾回收器回收的活动。

```

```

