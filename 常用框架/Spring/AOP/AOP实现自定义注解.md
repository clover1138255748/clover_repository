今天我们就来讲讲切点的另一种配置方式：@annotation，通过@annotation配置切点，我们可以灵活的控制切到哪个方法，同时可以进行一些个性化的设置，今天我们就用它来实现一个记录所有接口请求功能吧。



## 添加依赖

新建一个Spring Boot项目，打开pom.xml文件添加相关Maven依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 自定义一个注解

```
/**
 * @author duxiu
 * @create 2020-05-26 3:37 下午
 */
@Retention(RetentionPolicy.RUNTIME)//定义注解生命周期
@Target(ElementType.METHOD)//定义注解作用域
@Documented//表明这个注解能被javadoc记录
public @interface EagleEye {
    /**
     * 接口描述
     * @return
     */
    String desc() default "";

}
```

1. 定义了注解的生命周期为运行时
2. 定义了注解的作用域为方法
3. 标识该注解可以被JavaDoc记录
4. 定义注解名称为EagleEye（鹰眼，哈哈~~）
5. 定义一个元素desc，用来描述被修饰的方法

注解虽然定义好了，但是还用不了，因为没有具体的实现逻辑，接下来我们用AOP实现它。

## 配置切面

```
package com.runlion.wl.admin.common;

import com.esotericsoftware.minlog.Log;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;

/**
 * @author duxiu
 * @create 2020-05-26 3:44 下午
 */
@Aspect
@Component("logAspect")
public class EagleEyeAspect {

    // 配置织入点
    @Pointcut("@annotation(com.runlion.wl.admin.common.EagleEye)")
    public void eagleEyeAspect() {

    }

    // 具体增强
    @Around("eagleEye()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        long beforeLong = System.currentTimeMillis();
        ServletRequestAttributes servletRequestAttributes =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = servletRequestAttributes.getRequest();

        Signature signature = joinPoint.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;

        Method method = methodSignature.getMethod();
        EagleEye annotation = method.getAnnotation(EagleEye.class);
        String desc = annotation.desc();

        Log.info("方法增强前");

        // 原来的方法
        Object result = joinPoint.proceed();

        Log.info("方法增强后");

        long afterLong = System.currentTimeMillis();

        Log.info("方法耗时" + (afterLong - beforeLong));
        return result;
    }

}
```

## 帮你跳坑

这里启动报错了

```

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'feignConfigBeanPostProcessor' defined in file [/Users/caoliang/Project/WL/wl-common/common-core/target/classes/com/runlion/wl/common/space/FeignConfigBeanPostProcessor.class]: BeanPostProcessor before instantiation of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration': Initialization of bean failed; nested exception is java.lang.IllegalArgumentException: error at ::0 can't find referenced pointcut eagleEye
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:477)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:312)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:308)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.context.support.PostProcessorRegistrationDelegate.registerBeanPostProcessors(PostProcessorRegistrationDelegate.java:237)
	at org.springframework.context.support.AbstractApplicationContext.registerBeanPostProcessors(AbstractApplicationContext.java:703)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:528)
	at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107)
	at com.runlion.wl.admin.AdminWebApplication.main(AdminWebApplication.java:36)
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration': Initialization of bean failed; nested exception is java.lang.IllegalArgumentException: error at ::0 can't find referenced pointcut eagleEye
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:562)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:481)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:312)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:308)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:372)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1178)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1072)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:511)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:481)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:312)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:308)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.aop.framework.autoproxy.BeanFactoryAdvisorRetrievalHelper.findAdvisorBeans(BeanFactoryAdvisorRetrievalHelper.java:89)
	at org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator.findCandidateAdvisors(AbstractAdvisorAutoProxyCreator.java:102)
	at org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator.findCandidateAdvisors(AnnotationAwareAspectJAutoProxyCreator.java:88)
	at org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator.shouldSkip(AspectJAwareAdvisorAutoProxyCreator.java:103)
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.postProcessBeforeInstantiation(AbstractAutoProxyCreator.java:248)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation(AbstractAutowireCapableBeanFactory.java:1042)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.resolveBeforeInstantiation(AbstractAutowireCapableBeanFactory.java:1016)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:471)
	... 14 common frames omitted
Caused by: java.lang.IllegalArgumentException: error at ::0 can't find referenced pointcut eagleEye
	at org.aspectj.weaver.tools.PointcutParser.parsePointcutExpression(PointcutParser.java:319)
	at org.springframework.aop.aspectj.AspectJExpressionPointcut.buildPointcutExpression(AspectJExpressionPointcut.java:217)
	at org.springframework.aop.aspectj.AspectJExpressionPointcut.checkReadyToMatch(AspectJExpressionPointcut.java:190)
	at org.springframework.aop.aspectj.AspectJExpressionPointcut.getClassFilter(AspectJExpressionPointcut.java:169)
	at org.springframework.aop.support.AopUtils.canApply(AopUtils.java:220)
	at org.springframework.aop.support.AopUtils.canApply(AopUtils.java:279)
	at org.springframework.aop.support.AopUtils.findAdvisorsThatCanApply(AopUtils.java:311)
	at org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator.findAdvisorsThatCanApply(AbstractAdvisorAutoProxyCreator.java:119)
	at org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator.findEligibleAdvisors(AbstractAdvisorAutoProxyCreator.java:89)
	at org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator.getAdvicesAndAdvisorsForBean(AbstractAdvisorAutoProxyCreator.java:70)
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.wrapIfNecessary(AbstractAutoProxyCreator.java:346)
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.postProcessAfterInitialization(AbstractAutoProxyCreator.java:298)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization(AbstractAutowireCapableBeanFactory.java:421)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1635)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:553)
	... 36 common frames omitted


```

这里报错了差不多就2种情况

### 1-拼写错误

![image-20200526171938633](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200526171938633.png)

这里名字一样要对应上

2-jar包过低

这我是直接用了最新的

```
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.0</version>
</dependency>
```

![image-20200526172059523](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200526172059523.png)

反正error at ::0 can't find referenced pointcut XXX

就用以上方法解决就完事了

## 怎么使用自定义注解？

```
/**
 * @author duxiu
 * @create 2020-05-16 9:39 上午
 */
@RestController
public class BigScreenApiImpl implements BigScreenApi{
    @Autowired
    private BigScreenService bigScreenService;


    @Override
    @GetMapping("/admin/api/findBigScreenByUser")
    @EagleEye(desc = "aop简单实现")
    public BigScreenVO findBigScreenByUser() {
        return bigScreenService.findBigScreenByUser();
    }

    @Override
    @GetMapping("/admin/api/findBigScreenByBill")
    @EagleEye(desc = "aop简单实现")
    public BigScreenVO findBigScreenByBill() {
        return bigScreenService.findBigScreenByBill();
    }

    @Override
    @GetMapping("/admin/api/findBigScreenByFinance")
    @EagleEye(desc = "aop简单实现")
    public BigScreenVO findBigScreenByFinance() {
        return bigScreenService.findBigScreenByFinance();
    }
}
```

对于需要AOP增强的方法，我们只需要：

1. 在方法上加上@EagleEye注解
2. 通过desc元素设置方法的描述

## 效果

![image-20200526172323005](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200526172323005.png)

![image-20200526172340154](https://gitee.com/cdx_dayshow/picBed/raw/master/img/image-20200526172340154.png)

