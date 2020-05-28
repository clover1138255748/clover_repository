## 需求

需要记录哪个用户操作了哪个接口 

简单来说就是  假如某个业务员删除了一个用户!

我主管后面可以查到是哪个业务员操作的!

## 代码

直接上代码不BB

切面代码

```
package com.runlion.wl.admin.common;

import com.runlion.security.server.util.UserInfoUtil;
import com.runlion.wl.common.constants.ErrorCodeConstants;
import com.runlion.wl.common.exception.BaseException;
import com.runlion.wl.common.service.api.sequence.SequenceApi;
import com.runlion.wl.common.util.AddressUtil;
import com.runlion.wl.common.vo.UserInfoVO;
import com.runlion.wl.user.api.identity.IdentityApi;
import com.runlion.wl.user.vo.admin.identity.IdentityVO;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.annotation.Order;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

import javax.servlet.http.HttpServletRequest;
import java.util.Date;

/**
 * @author duxiu
 * @create 2020-05-28 10:12 上午
 */
@Aspect
@Component
@Order(1)
@Slf4j
public class SystemLogAspect {

    @Autowired
    private TaskExecutor taskExecutor;
    @Autowired
    private IdentityApi identityApi;
    @Autowired
    private SequenceApi sequenceApi;

    /***
     * 定义controller切入点拦截规则，拦截SystemControllerLog注解的方法
     */
    @Pointcut("execution(* com.runlion.wl.*.controller.*.*.*(..))")
    public void controllerAspect() {}

    /***
     * 拦截控制层的操作日志
     * 
     * @param joinPoint
     * @return
     */
    @Before("controllerAspect()")
    public void recordLog(JoinPoint joinPoint) {
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = (HttpServletRequest) requestAttributes.resolveReference(RequestAttributes.REFERENCE_REQUEST);
        taskExecutor.execute(() -> {
            try {
                SystemLog systemLog = new SystemLog();
                systemLog.BuildId(sequenceApi);
                // 请求方法UserId
                String userId = getSessionId();
                systemLog.setUserId(userId);
                IdentityVO identityVO = identityApi.queryByUserId(userId);
                if (identityVO != null) {
                    systemLog.setRealName(identityVO.getName());
                }
                // Ip
                String ip = AddressUtil.getLocalIP();
                systemLog.setIp(ip);
                // eureka模块
                int localPort = request.getLocalPort();
                systemLog.setEurekaName(EurekaPortEnum.getEurekaPortEnumKey(localPort));
                // 方法名
                String method = joinPoint.getTarget().getClass().getName() + "."
                        + joinPoint.getSignature().getName();
                systemLog.setActionMethod(method);
                //创建日期
                systemLog.setGmtCreate(new Date());

                System.out.println("插入数据库systemLog=" + systemLog);
            } catch (Exception e2) {
                log.error("新增日志失败：" + e2.getMessage(), e2);
                throw new BaseException("", ErrorCodeConstants.FEE_TEMPLATE_IN_USE);
            }
        });
    }

    /**
     * 获取登录的用户id
     *
     * @return
     */
    protected String getSessionId() {
        try {
            final UserInfoVO user = UserInfoUtil.getCurrentUser(UserInfoVO.class);
            if (user == null) {
                throw new BaseException(ErrorCodeConstants.CHECK_USER_NOT_LOGIN.getDes());
            }
            return user.getUserId();
        } catch (Exception e) {
            throw new BaseException(ErrorCodeConstants.CHECK_USER_NOT_LOGIN.getDes(), e);
        }
    }


}
```

插入到数据库的DO

```
package com.runlion.wl.admin.common;

import com.runlion.wl.common.constants.SequenceNameDefineEnum;
import com.runlion.wl.common.service.api.sequence.SequenceApi;
import com.runlion.wl.common.util.IdUtil;
import lombok.Data;

import java.util.Date;

/**
 * @author duxiu
 * @create 2020-05-28 13:41
 */
@Data
public class SystemLog {
    private String id;

    /**
     * userId
     */
    private String userId;
    /**
     * 用户姓名
     */
    private String realName;
    /**
     * 方法名
     */
    private String actionMethod;
    /**
     * ip
     */
    private String ip;
    /**
     * 微服务名称
     *
     */
    private String eurekaName;


    /**
     * 创建日期
     */
    private Date gmtCreate;


    public void BuildId(SequenceApi sequenceApi) {
        String id = IdUtil.getId(SequenceNameDefineEnum.SYSTEM_LOG,
                sequenceApi.nextValue(SequenceNameDefineEnum.SYSTEM_LOG.getSequenceName()));
        this.id = id;
    }
}
```

//EurekaNameEnum

```
package com.runlion.wl.admin.common;

/**
 * @author duxiu
 * @create 2020-05-28 15:14
 */
public enum EurekaPortEnum {
    /**
     * admin
     */
    ADMIN(7779, "ADMIN"),

    /**
     * bill
     */
    BILL(7771, "BILL" ),

    /**
     * user
     */
    USER(7775, "USER" ),

    /**
     * thirdpt
     */
    THIRDPT(7772, "THIRDPT" ),
    /**
     * finance
     */
    FINANCE(7778, "FINANCE"),

    /**
     * fund
     */
    FUND(7788, "FUND" );

    private final int port;

    private final String key;


    EurekaPortEnum(int port, String key) {
        this.port = port;
        this.key = key;
    }

    public int getPort() {
        return port;
    }

    public String getKey() {
        return key;
    }




    public static String getEurekaPortEnumKey(int port) {
        for (EurekaPortEnum tmp : EurekaPortEnum.values()) {
            if (tmp.getPort() == port) {
                return tmp.getKey();
            }
        }
        return "";
    }

    public static EurekaPortEnum parse(int port) {
        for (EurekaPortEnum value : values()) {
            if (value.getPort() == port) {
                return value;
            }
        }
        return null;
    }

}
```



这里我是在方法进来之前执行了这个操作

然后我直接放到一个异步执行器里面

这样是为了不影响接口性能

这枚举的作用是我能根据request 中的Port判断这个接口来自哪个模块

这个作用是为了统计下哪个微服务被调用接口的次数

这里还需要注意的点是ip地址的获取

因为前端访问我们都是通过网关代理 的

所以我们获取ip要从request中获取

具体代码贴上了

```
public static String getLocalIpByRequest(HttpServletRequest request) {
    // 获取请求主机IP地址,如果通过代理进来，则透过防火墙获取真实IP地址
    String headerName = "x-forwarded-for";
    String ip = request.getHeader(headerName);
    if (null != ip && ip.length() != 0 && !"unknown".equalsIgnoreCase(ip)) {
        // 多次反向代理后会有多个IP值，第一个IP才是真实IP,它们按照英文逗号','分割
        if (ip.indexOf(",") != -1) {
            ip = ip.split(",")[0];
        }
    }
    if (checkIp(ip)) {
        headerName = "Proxy-Client-IP";
        ip = request.getHeader(headerName);
    }
    if (checkIp(ip)) {
        headerName = "WL-Proxy-Client-IP";
        ip = request.getHeader(headerName);
    }
    if (checkIp(ip)) {
        headerName = "HTTP_CLIENT_IP";
        ip = request.getHeader(headerName);
    }
    if (checkIp(ip)) {
        headerName = "HTTP_X_FORWARDED_FOR";
        ip = request.getHeader(headerName);
    }
    if (checkIp(ip)) {
        headerName = "X-Real-IP";
        ip = request.getHeader(headerName);
    }
    if (checkIp(ip)) {
        headerName = "remote addr";
        ip = request.getRemoteAddr();
        // 127.0.0.1 ipv4, 0:0:0:0:0:0:0:1 ipv6
        if ("127.0.0.1".equals(ip) || "0:0:0:0:0:0:0:1".equals(ip)) {
            //根据网卡取本机配置的IP
            InetAddress inet = null;
            try {
                inet = InetAddress.getLocalHost();
            } catch (UnknownHostException e) {
                e.printStackTrace();
            }
            ip = inet.getHostAddress();
        }
    }
    return ip;
}
private static boolean checkIp(String ip) {
    if (null == ip || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        return true;
    }
    return false;
}
```