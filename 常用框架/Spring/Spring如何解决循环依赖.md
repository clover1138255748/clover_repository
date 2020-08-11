# 什么是循环依赖？

循环依赖其实就是对象之间的循环引用，即两个或两个以上的Bean互相持有对方，最终形成闭环。





![image-20200721172905644](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200721172905644.png)

```
public class ClassA {
  private ClassB classB;
  public ClassB getClassB() {    return classB;  }
  public void setClassB(ClassB classB) {    this.classB = classB;  }}
public class ClassB {
  private ClassA classA;
  public ClassA getClassA() {    return classA;  }
  public void setClassA(ClassA classA) {    this.classA = classA;  }}
```



# 循环依赖处理机制？



- **单例bean构造器参数方式循环依赖（无法解决）**

首先先来演示通过构造器参数方式的循环依赖，结合源码来分析，为什么构造器参数的方式无法解决循环依赖。

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUgmxNQyAlXYW8wvktmkGlKmJFLPBS6FYMqODw97sruYzuia7AFwSEvkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`getBean()`方法

```
public Object getBean(String name) throws BeansException {  return doGetBean(name, null, null, false);}
```

进入到`doGetBean()`方法

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUic3SibaqR6GcXJ4OALhvuCvGiaDOw7KHIppGSKGK0C8RnEbtIiaZriaCicjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先会尝试从缓存中获取对象，此时还没有这个对象，接着往下看     ![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUAVqR1X7UjSBUMGUxEO6yUCs6cpcHPwO21C85OwaB66gHMaZtiaPtg9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRU0ibU11kRxpOIicYmfscsP5ZOUUXMVzlP8tCRibtiamMzfoPoUvrY9xsNmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里会将每一个正在创建的BeanName放入`singletonsCurrentlyInCreation`集合中，该集合在`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`中定义。

```
private final Set<String> singletonsCurrentlyInCreation =      Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

接着往下走，执行到`singletonObject = singletonFactory.getObject()`，会进入到前面的lamda表达式中的`createBean`方法中，来到`doCreateBean`方法中并进入。

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUhS2v0avVEmIe1317icUPvcStb50JS5S8u3RtsOLrD8DbldRo2BvkoxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`doCreateBean`方法中，这里会有几个比较重要的几个部分，此步骤是将对象实例化。

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUIjibpl6BxxVlEicA8stfOKZjPtKqWmpbKsXgP0G9TPYb04pjaGcvZK7w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下将实例化的对象提前暴露到`singletonFactories（也就是三级缓存）`中

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUpVcno595dHDVpcIiaGLb14u4vpdW6ooRvSDOyMY7P5ibYcSw3p2wMibIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`addSingletonFactory`方法里

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUufKlfBqfQDibtq8f72qIzwCodLsJIBhDvTMrV4CLr2zZR6XZmgpR6Bw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接着往下执行，来到`populateBean`方法，这里就是判断当前bean是否依赖了其他的bean，如果依赖了，就会递归的调用getBean()方法尝试获取目标bean

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUibFzaicyTsvSVicBXIWUf6SnOo0nctnfggMUib7NZgeiaTh2J1ZrSojPXxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入方法内部，执行到`applyPropertyValues`部分

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUjpUibibDC2uZ4PdaicQDqlibic9lQxnCaniaFAQK3iazruUq99vjZ1icViaq0GQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`applyPropertyValues`方法内部

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUVkAgtQib8HlbqHLBDQeI2bPHZxhp7hUldGq5C2J6l1V4NiaicZhKhl2BQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUVtrCb7iaZiaTtuNemXx38rTpC7GvYS2ibrwLboC8eFl6ibvATlJsEoHquw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来就是重点了，进入方法内部，找到`bean = this.beanFactory.getBean(refName)`这行代码。

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUOyzO5F1UBOz3diaB8STJibeBJl4sMk6HZrLWA1TXOziaCqV0NMyljN4EQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里发现ClassA对象依赖于ClassB对象，又回到`getBean()`方法，但是要注意的是，此时getBean里的参数是classB，此时容器中还不存在ClassB这个对象，又回到创建ClassB的过程，执行到属性装配的步骤时，ClassB对象又依赖于ClassA对象，又回到之前的步骤，执行到`getSingleton`中的`beforeSingletonCreation(beanName)`方法时，发现此时ClassA已经存在`singletonsCurrentlyInCreation`这个集合中了，接着会抛出异常，整个过程结束。（这也是为什么构造器参数形式不支持循环依赖的原因）

```
Error creating bean with name 'classA' defined in class path resource [applicationContext.xml]: Cannot resolve reference to bean 'classB' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classB' defined in class path resource [applicationContext.xml]: Cannot resolve reference to bean 'classA' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'classA': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

- **prototype原型bean方式循环依赖（无法解决）**





![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUGx1ZyzgFO5ibARA5qYL3Ev0c6cyFGqOTk3WRQicmU0kn0AkTBWnXlzZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ClassA对象在创建之前，会进行标记这个bean正在被创建，等创建之后会将标记删除。

进入`beforePrototypeCreation`方法，看一下此方法都做了哪些事。

![img](https://mmbiz.qpic.cn/mmbiz_png/16XqH2UnibgLJHWxssEuPPDP2YGznUgRUbgWKZicpyia4oygmzzxRUIkk3YjRXY0FL6O1QnR1zNR7tvWmdujPxwBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里可以看到，`prototypesCurrentlyInCreation`是一个ThreadLocal，ThreadLocal是什么呢？其实它就是一个线程局部变量，它的功能就是为每一个使用该变量的线程都提供一个变量值的副本，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。也就是说，此处会将beanName存放到线程局部变量中，后面创建对象的时候，首先会查看这个局部变量是否有值，如果没有值，说明这个对象还没有进行创建，会进入到创建对象的步骤，如果检测到这个beanName已存在，就会抛出异常。这也是为什么prototype原型bean方式循环依赖无法解决的原因。

```
Error creating bean with name 'classA' defined in class path resource [applicationContext.xml]: Cannot resolve reference to bean 'classB' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classB' defined in class path resource [applicationContext.xml]: Cannot resolve reference to bean 'classA' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'classA': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

- **singleton单例bean的setter方式循环依赖**

而singleton单例bean的settter方式的循环依赖，它是通过提前暴露一个ObjectFactory对象来完成的，简单来说ClassA在调用构造器完成对象初始化之后，在调用ClassA的setClassB方法之前就把ClassA实例化的对象通过ObjectFactory提前暴露到Spring容器中，ClassA调用setClassB方法，Spring首先尝试从容器中获取ClassB，此时ClassB不存在，Spring容器初始化ClassB，同时也将ClassB暴露到Spring容器中，ClassB调用setClassA方法，Spring从容器中获取ClassA，因为之前已经提前暴露了ClassA，因此可以获取到ClassA实例，ClassA通过给Spring容器获取到ClassB，完成对象初始化操作，这样ClassA和ClassB都完成了对象初始化操作，解决了循环依赖问题。



# Spring是如何解决循环依赖的？

Spring正是通过singleton单例bean的setter方式解决循环依赖的，ClassA在创建的时候，会将自己放入到singletonObjectFactories三级缓存中，在属性装配的时候，检测到ClassA对象依赖于ClassB对象，Spring首先尝试从容器中获取ClassB，此时ClassB不存在，Spring容器会初始化ClassB，同时也将ClassB暴露到容器中，接着就是对ClassB进行属性装配，装配过程中发现依赖于ClassA，于是Spring从容器中获取ClassA，因为之前已经将ClassA暴露到容器中了，因此可以获取到ClassA实例，并将ClassA升级放入二级缓存earlySingletonObjects中，ClassA通过给Spring容器获取到ClassB，完成了ClassB的初始化操作，这样ClassA和ClassB都完成了对象初始化操作，如此便解决了循环依赖问题。

------

ClassA升级放入earlySingletonObjects的代码（此步骤是在ClassB进行属性装配时，发现ClassA已经存在三级缓存中，于是取出放入二级缓存）

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {  Object singletonObject = this.singletonObjects.get(beanName);  // 判断当前单例bean是否正在创建中  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {    synchronized (this.singletonObjects) {      singletonObject = this.earlySingletonObjects.get(beanName);      // 是否允许从singletonFactories中通过getObject拿到对象      if (singletonObject == null && allowEarlyReference) {        // 从三级缓存中取出单例对象工厂        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);        if (singletonFactory != null) {          singletonObject = singletonFactory.getObject();          // 从三级缓存移动到了二级缓存          this.earlySingletonObjects.put(beanName, singletonObject);          this.singletonFactories.remove(beanName);        }      }    }  }  return singletonObject;}
```



ClassB初始化完成放入单例池的代码

```
protected void addSingleton(String beanName, Object singletonObject) {  synchronized (this.singletonObjects) {  // 将此bean（已成型的Spring Bean）移入一级缓存（单例池）    this.singletonObjects.put(beanName, singletonObject);    this.singletonFactories.remove(beanName);    this.earlySingletonObjects.remove(beanName);    this.registeredSingletons.add(beanName);  }}
```

那么完成了对ClassB的初始化操作，这样ClassA和ClassB都完成了对象初始化操作。ClassA形成最终的Spring Bean对象之后，会将其放入单例池中，代码同上。

到这里，Spring整个解决循环依赖问题的实现思路已经比较清楚了。对于整体过程，读者朋友只要理解两点：

- **Spring是通过递归的方式获取目标bean及其所依赖的bean的；**

- **Spring实例化一个bean的时候，是分两步进行的，首先实例化目标bean，然后为其注入属性。**

# 总结

也就是说，Spring在实例化一个bean的时候，是首先递归的实例化其所依赖的所有bean，直到某个bean没有依赖其他bean，此时就会将该实例返回，然后反递归的将获取到的bean设置为各个上层bean的属性的。











