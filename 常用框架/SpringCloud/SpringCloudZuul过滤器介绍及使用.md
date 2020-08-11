## 过滤器类型

Zuul 中的过滤器跟我们之前使用的 javax.servlet.Filter 不一样，javax.servlet.Filter 只有一种类型，可以通过配置 urlPatterns 来拦截对应的请求。

而 Zuul 中的过滤器总共有 4 种类型，且每种类型都有对应的使用场景。

#### 1）pre

可以在请求被路由之前调用。适用于身份认证的场景，认证通过后再继续执行下面的流程。

#### 2）route

在路由请求时被调用。适用于灰度发布场景，在将要路由的时候可以做一些自定义的逻辑。

#### 3）post

在 route 和 error 过滤器之后被调用。这种过滤器将请求路由到达具体的服务之后执行。适用于需要添加响应头，记录响应日志等应用场景。

#### 4）error

处理请求时发生错误时被调用。在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息。

## 请求生命周期

可以通过图 1 看出整个过滤器的执行生命周期，此图来自 Zuul GitHub wiki 主页。

![过滤器生命周期](https://gitee.com/cdx_dayshow/picBed/raw/master/img/5-1ZR31034432N.png)
		                                                                           图 1 过滤器生命周期


通过上面的图可以清楚地知道整个执行的顺序，请求发过来首先到 pre 过滤器，再到 routing 过滤器，最后到 post 过滤器，任何一个过滤器有异常都会进入 error 过滤器。

通过 com.netflix.zuul.http.ZuulServlet也可以看出完整执行顺序，ZuulServlet 类似 Spring-Mvc 的 DispatcherServlet，所有的 Request 都要经过 ZuulServlet 的处理。

ZuulServlet 源码如下所示：

```
@Override
public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse)
        throws ServletException, IOException {
    try {
        init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);
        RequestContext context = RequestContext.getCurrentContext();
        context.setZuulEngineRan();
        try {
            preRoute();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            route();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            postRoute();
        } catch (ZuulException e) {
            error(e);
            return;
        }
    } catch (Throwable e) {
        error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
    } finally {
        RequestContext.getCurrentContext().unset();
    }
}
```

由代码可知，自定义过滤器需要继承 ZuulFilter，并且需要实现下面几个方法：

#### 1）shouldFilter

是否执行该过滤器，true 为执行，false 为不执行，这个也可以利用配置中心来实现，达到动态的开启和关闭过滤器。

#### 2）filterType

过滤器类型，可选值有 pre、route、post、error。

#### 3）filterOrder

过滤器的执行顺序，数值越小，优先级越高。

#### 4）run

执行自己的业务逻辑，本段代码中是通过判断请求的 IP 是否在黑名单中，决定是否进行拦截。blackIpList 字段是 IP 的黑名单，判断条件成立之后，通过设置 ctx.setSendZuulResponse（false），告诉 Zuul 不需要将当前请求转发到后端的服务了。通过 setResponseBody 返回数据给客户端。

## 过滤器禁用

有的场景下，我们需要禁用过滤器，此时可以采取下面的两种方式来实现：

- 利用 shouldFilter 方法中的 return false 让过滤器不再执行
- 通过配置方式来禁用过滤器，格式为“zuul. 过滤器的类名.过滤器类型 .disable=true”。如果我们需要禁用“使用过滤器”部分中的 IpFilter，可以用下面的配置：

```
zuul.IpFilter.pre.disable=true
```

## 过滤器中传递数据

项目中往往会存在很多的过滤器，执行的顺序是根据 filterOrder 决定的，那么肯定有一些过滤器是在后面执行的，如果你有这样的需求：第一个过滤器需要告诉第二个过滤器一些信息，这个时候就涉及在过滤器中怎么去传递数据给后面的过滤器。

实现这种传值的方式笔者第一时间就想到了用 ThreadLocal，既然我们用了 Zuul，那么 Zuul 肯定有解决方案，比如可以通过 RequestContext 的 set 方法进行传递，RequestContext 的原理就是 ThreadLocal。

RequestContext ctx = RequestContext.getCurrentContext();
ctx.set("msg", "你好吗");

后面的过滤就可以通过 RequestContext 的 get 方法来获取数据：

RequestContext ctx = RequestContext.getCurrentContext();
ctx.get("msg");

上面我们说到 RequestContext 的原理就是 ThreadLocal，这不是随便说的，而是看过源码得出来的结论，下面请看源码，代码如下所示。

```
protected static final ThreadLocal<? extends RequestContext> threadLocal = new ThreadLocal<RequestContext>() {
    @Override
    protected RequestContext initialValue() {
        try {
            return contextClass.newInstance();
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
    }
};
public static RequestContext getCurrentContext() {
    if (testContext != null)
        return testContext;
    RequestContext context = threadLocal.get();
    return context;
}
```

## 过滤器中异常处理

对于异常来说，无论在哪个地方都需要处理。过滤器中的异常主要发生在 run 方法中，可以用 try catch 来处理。Zuul 中也为我们提供了一个异常处理的过滤器，当过滤器在执行过程中发生异常，若没有被捕获到，就会进入 error 过滤器中。

我们可以定义一个 error 过滤器来记录异常信息，代码如下所示。

```
public class ErrorFilter extends ZuulFilter {
    private Logger log = LoggerFactory.getLogger(ErrorFilter.class);
    @Override
    public String filterType() {
        return "error";
    }
    @Override
    public int filterOrder() {
        return 100;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        Throwable throwable = ctx.getThrowable();
        log.error("Filter Erroe : {}", throwable.getCause().getMessage());
        return null;
    }
}
```

 下面是记录日志代码

```
package com.runlion.wl.gate.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import com.runlion.security.server.util.UserInfoUtil;
import com.runlion.wl.common.constants.ErrorCodeConstants;
import com.runlion.wl.common.exception.BaseException;
import com.runlion.wl.common.util.AddressUtil;
import com.runlion.wl.common.util.date.DateFormatterEnum;
import com.runlion.wl.common.util.date.DateUtil;
import com.runlion.wl.common.vo.UserInfoVO;
import com.runlion.wl.gate.aspect.SystemLog;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.util.Date;

/**
 * @author duxiu
 * @create 2020-06-06 13:41
 */
@Component
@Slf4j
public class SystemLogFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return "p";
    }

    @Override
    public int filterOrder() {
        return 2;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext() ;
        HttpServletRequest request = ctx.getRequest() ;
        try {
            SystemLog systemLog = new SystemLog();
            // 请求方法UserId
            String userId = getSessionId();
            systemLog.setUserId(userId);
            // Ip
            String ip = AddressUtil.getLocalIpByRequest(request);
            systemLog.setIp(ip);
            // 方法名
            StringBuffer method =request.getRequestURL();
            systemLog.setActionMethod(method.toString());
            //创建日期
            systemLog.setGmtCreate(DateUtil.format(new Date(), DateFormatterEnum.DATE_WITH_TIME.getCode()));
            log.info("******************************************************************************************");
            log.info("操作日志:"+systemLog.toString());
            log.info("******************************************************************************************");
        } catch (Exception e) {
            log.error("新增日志失败：" + e.getMessage(), e);
            throw new BaseException("新增日志失败", ErrorCodeConstants.FEE_TEMPLATE_IN_USE);
        }
        return null;
    }

    private String getSessionId() {
        try {
            final UserInfoVO user = UserInfoUtil.getCurrentUser(UserInfoVO.class);
            if (user == null) {
                //throw new BaseException(ErrorCodeConstants.CHECK_USER_NOT_LOGIN.getDes());
                return "无用户接口";
            }
            return user.getUserId();
        } catch (Exception e) {
            throw new BaseException(ErrorCodeConstants.CHECK_USER_NOT_LOGIN.getDes(), e);
        }
    }
}

```

