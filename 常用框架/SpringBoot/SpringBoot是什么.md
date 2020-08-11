## **什么是springboot**

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域（rapid application development）成为领导者。

spring大家都知道，boot是启动的意思。所以，spring boot其实就是一个启动spring项目的一个工具而已。从最根本上来讲，Spring Boot就是一些库的集合，它能够被任意项目的构建系统所使用。


## **为什么会出现**

以前在写spring项目的时候，要配置各种xml文件，还记得曾经被ssh框架支配的恐惧。随着spring3，spring4的相继推出，约定大于配置逐渐成为了开发者的共识，大家也渐渐的从写xml转为写各种注解，在spring4的项目里，你甚至可以一行xml都不写。

虽然spring4已经可以做到无xml，但写一个大项目需要茫茫多的包，maven配置要写几百行，也是一件很可怕的事。

现在，快速开发一个网站的平台层出不穷，nodejs，php等虎视眈眈，并且脚本语言渐渐流行了起来（Node JS，Ruby，Groovy，Scala等），spring的开发模式越来越显得笨重。

在这种环境下，spring boot伴随着spring4一起出现了。

## **可以做什么**

那么，spring boot可以做什么呢？

spring boot并不是一个全新的框架，它不是spring解决方案的一个替代品，而是spring的一个封装。所以，你以前可以用spring做的事情，现在用spring boot都可以做。

现在流行微服务与分布式系统，springboot就是一个非常好的微服务开发框架，你可以使用它快速的搭建起一个系统。同时，你也可以使用spring cloud（Spring Cloud是一个基于Spring Boot实现的云应用开发工具）来搭建一个分布式的网站。


## **优点**

### 使编码变得简单

spring boot采用java config的方式，对spring进行配置，并且提供了大量的注解，极大地提高了工作效率。

### 使配置变得简单

spring boot提供许多默认配置，当然也提供自定义配置。但是所有spring boot的项目都只有一个配置文件：application.properties/application.yml。用了spring boot，再也不用担心配置出错找不到问题所在了。

### 使部署变得简单

spring boot内置了三种servlet容器：tomcat，jetty，undertow。

所以，你只需要一个java的运行环境就可以跑spring boot的项目了。spring boot的项目可以打成一个jar包，然后通过`java -jar xxx.jar`来运行。（spring boot项目的入口是一个main方法，运行该方法即可。 ）

### 使监控变得简单

spring boot提供了actuator包，可以使用它来对你的应用进行监控。它主要提供了以下功能：

