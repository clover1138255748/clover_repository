JDK1.8学习笔记



- [JDK1.8主要更新的内容](#jdk18-------)
  * [Lambda表达式](#lambda---)
    + [构造器引用](#-----)
    + [数组引用](#----)
  * [了解Stream](#--stream)
  * [Stream 的操作三个步骤](#stream--------)
    + [1-创建 Stream](#1----stream)
    + [Stream 的终止操作](#stream------)
  * [并行流与串行流](#-------)

## JDK1.8主要更新的内容

1. Lambda 表达式
2. 函数式接口
3. 方法引用与构造器引用
4. Stream API
5. 接口中的默认方法与静态方法
6. 新时间日期API
7. 其他新特性

- 速度更快
- 代码更少（增加了新的语法 **Lambda** **表达式**）
- 强大的 **Stream** **API**
- 便于并行
- 最大化减少空指针异常 Optional

我们主要针对几点学习

### Lambda表达式

Lambda 是一个匿名函数，我们可以把 Lambda 表达式理解为是一段可以传递的代码（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升

Lambda 表达式在Java 语言中引入了一个新的语法元素和操作符。这个操作符为 “->” ， 该操作符被称为 Lambda 操作符或剪头操作符。它将 Lambda 分为两个部分：

\* 左侧：指定了 Lambda 表达式需要的所有参数

\* 右侧：指定了 Lambda 体，即 Lambda 表达式要执行的功能。

\## 表达式语法

\* 语法格式一：无参数，无返回值

```
() -> System.out.println("Hello Lambda!");
```

\* 语法格式二：有一个参数，并且无返回值

```
(x) -> System.out.println(x)
```

\* 语法格式三：若只有一个参数，小括号可以省略不写

```
x -> System.out.println(x)
```

\* 语法格式四：有两个以上的参数，有返回值，并且 Lambda 体中有多条语句

```
Comparator<Integer> com = (x, y) -> {

System.out.println("函数式接口");

return Integer.compare(x, y);

};
```

\* 语法格式五：若 Lambda 体中只有一条语句， return 和 大括号都可以省略不写

```
Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
```

\* 语法格式六：Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出，数据类型，即“类型推断”

(Integer x, Integer y) -> Integer.compare(x, y);

上联：左右遇一括号省

下联：左侧推断类型省

横批：能省则省

类型推断

上述 Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的 “类型推断”

\## 函数式接口

Lambda 表达式需要“函数式接口”的支持

函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。 可以使用注解 @FunctionalInterface 修饰可以检查是否是函数式接口

\* 只包含一个抽象方法的接口，称为函数式接口。

\* 你可以通过 Lambda 表达式来创建该接口的对象。（若 Lambda 表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方 法上进行声明）。

\* 我们可以在任意函数式接口上使用 @FunctionalInterface 注解， 这样做可以检查它是否是一个函数式接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。

**作为参数传递 Lambda 表达式：为了将 Lambda 表达式作为参数传递，接收Lambda 表达式的参数类型必须是与该 Lambda 表达式兼容的函数式接口的类型。**



```
* Consumer<T> : 消费型接口

void accept(T t);

\* Supplier<T> : 供给型接口

T get();

\* Function<T, R> : 函数型接口

R apply(T t);

\* Predicate<T> : 断言型接口

boolean test(T t);
```

\## 方法引用与构造器引用

\### 方法引用

当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！

（实现抽象方法的参数列表，必须与方法引用方法的参数列表保持一致！）

方法引用：使用操作符 “::” 将方法名和对象或类的名字分隔开来。

如下三种主要使用情况：

* 对象::实例方法

![image-20200509163212107](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163212107.png)

* 类::静态方法

![image-20200509163257548](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163257548.png)

* 类::实例方法

![image-20200509163310922](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163310922.png)

注意：当需要引用方法的第一个参数是调用对象，并且第二个参数是需要引

用方法的第二个参数(或无参数)时：ClassName::methodName

####  构造器引用

格式： ClassName::new

与函数式接口相结合，自动与函数式接口中方法兼容。 可以把构造器引用赋值给定义的方法，与构造器参数列表要与接口中抽象方法的参数列表一致！

![image-20200509163333944](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163333944.png)

#### 数组引用

格式： type[] :: new

![image-20200509163345409](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163345409.png)

### 了解Stream

Java8中有两大最为重要的改变。第一个是 Lambda 表达式；另外一个则是 Stream API(java.util.stream.*)。
Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。 使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简而言之， Stream API 提供了一种高效且易于使用的处理数据的方式。

流(Stream) 到底是什么呢？
是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。
“集合讲的是数据，流讲的是计算！”

注意：
①Stream 自己不会存储元素。
②Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
③Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

### Stream 的操作三个步骤

#### 1-创建 Stream

#### ![image-20200509163850483](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509163850483.png)2-中间操作

筛选与切片
filter——接收 Lambda ， 从流中排除某些元素。
limit——截断流，使其元素不超过给定数量。
skip(n) —— 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补
distinct——筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素

- filter 示例
  ![image-20200509164011988](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164011988.png)

- map 示例
  ![image-20200509164022844](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164022844.png)

#### Stream 的终止操作

- allMatch——检查是否匹配所有元素

- anyMatch——检查是否至少匹配一个元素

- noneMatch——检查是否没有匹配的元素

- findFirst——返回第一个元素

- findAny——返回当前流中的任意元素

- count——返回流中元素的总个数

- max——返回流中最大值

- min——返回流中最小值

- forEach 遍历
  前面都是字面意思没什么好说的
  下面才是重点

- reduce 归约

  ![image-20200509164106082](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164106082.png)

![image-20200509164150995](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164150995.png)



最常用的还是这个

* 收集 collector

![image-20200509164258224](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164258224.png)

![image-20200509164324883](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164324883.png)

![image-20200509164342838](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164342838.png)

### 并行流与串行流

并行流就是把一个内容分成多个数据块，并用不同的线程分
别处理每个数据块的流。

Java 8 中将并行进行了优化，我们可以很容易的对数据进行并行操作。Stream API 可以声明性地通过 parallel() 与sequential() 在并行流与顺序流之间进行切换。

- 了解 Fork/Join 框架
  ![image-20200509164415500](http://qa0a9jn2t.bkt.clouddn.com/img/image-20200509164415500.png)
- Fork/Join 框架与传统线程池的区别
  采用 “工作窃取”模式（work-stealing）：
  当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线 程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。

相对于一般的线程池实现,fork/join框架的优势体现在对其中包含的任务的处理方式上.在一般的线程池中,如果一个线程正在执行的任务由于某些原因无法继续运行,那么该线程会处于等待状态.而在fork/join框架实现中,如果某个子问题由于等待另外一个子问题的完成而无法继续运行.那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程的等待时间,提高了性能.