---
layout:     post
title:      "Spring Boot AOP记录用户操作日志"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-07 21:31:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - AOP
    - 日志
---
在Spring框架中，使用AOP配合自定义注解可以方便的实现用户操作的监控。创建一个含`JdbcTemplate`的Spring Boot应用，可以参考[SpringBoot 中使用JdbcTemplate](https://tengxt.gitee.io/2019/08/01/Spring-Boot%E4%B8%AD%E4%BD%BF%E7%94%A8-JdbcTemplate/)

#### 引入aop的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

#### 自定义注解

定义一个方法级别的`@Log`注解，用于标注要监控的方法

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

#### 创建表

```sql
CREATE TABLE `sys_log` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL COMMENT '用户名',
  `operation` varchar(50) DEFAULT NULL COMMENT '用户操作',
  `time` varchar(50) DEFAULT NULL COMMENT '响应时间',
  `method` varchar(255) DEFAULT NULL COMMENT '请求方法',
  `ip` varchar(50) DEFAULT NULL COMMENT 'IP地址',
  `createTime` date DEFAULT NULL COMMENT '创建时间',
  `params` varchar(255) DEFAULT NULL COMMENT '请求参数',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```

#### 保存日志的方法

定义一个`SysLogService`接口，包含一个保存操作日志的抽象方法

```java
import com.tengxt.springbootaoplog.entity.SysLog;

public interface SysLogService {
    int saveSysLog(SysLog sysLog);
}
```

其接口对应的实现方法

```java
import com.tengxt.springbootaoplog.entity.SysLog;
import com.tengxt.springbootaoplog.service.SysLogService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class SysLogServiceImpl implements SysLogService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public int saveSysLog(SysLog sysLog) {
        StringBuffer sql = new StringBuffer("INSERT INTO sys_log(username,operation,time,method,ip,createTime,params)");
        sql.append("VALUES(:username,:operation,:time,:method,:ip,:createTime,:params)");

        NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(this.jdbcTemplate.getDataSource());
        return namedParameterJdbcTemplate.update(sql.toString(), new BeanPropertySqlParameterSource(sysLog));
    }
}
```

#### 切面和切点

定义一个`LogAspect`类，使用`@Aspect`标注让其成为一个切面，切点为使用`@Log`注解标注的方法，使用`@Around`环绕通知

```java

import com.tengxt.springbootaoplog.entity.SysLog;
import com.tengxt.springbootaoplog.service.SysLogService;
import com.tengxt.springbootaoplog.util.HttpContextUtils;
import com.tengxt.springbootaoplog.util.IPUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.LocalVariableTableParameterNameDiscoverer;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.util.Date;

@Aspect
@Component
public class LogAspect {

    @Autowired
    private SysLogService sysLogService;

    @Pointcut("@annotation(com.tengxt.springbootaoplog.config.Log)")
    public void pointcut(){}

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint point){
        Object result = null;
        long beginTime = System.currentTimeMillis();
        try {
            // 执行方法
            result = point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        // 执行时长（毫秒）
        long endTime = System.currentTimeMillis();
        // 保存日志
        saveLog(point, (endTime - beginTime));
        return result;
    }

    private void saveLog(ProceedingJoinPoint joinPoint, Long time){
        MethodSignature signature = (MethodSignature)joinPoint.getSignature();
        Method method = signature.getMethod();
        SysLog sysLog = new SysLog();
        Log logAnnotation = method.getAnnotation(Log.class);
        if(null != logAnnotation){
            // 注解上的描述
            sysLog.setOperation(logAnnotation.value());
        }
        // 请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        sysLog.setMethod(className+"."+methodName+"()");
        // 请求的方法参数值
        Object[] args = joinPoint.getArgs();
        // 请求的方法参数名称
        LocalVariableTableParameterNameDiscoverer nameDiscoverer = new LocalVariableTableParameterNameDiscoverer();
        String[] parameterNames = nameDiscoverer.getParameterNames(method);
        if(null != args && null != parameterNames){
            String params = "";
            for(int i = 0; i < args.length; i++){
                params += " " + parameterNames[i] + " : " + args[i];
            }
            sysLog.setParams(params);
        }
        // 获取request
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        // 设置IP地址
        sysLog.setIp(IPUtils.getIpAddr(request));
        // 模拟一个用户名
        sysLog.setUsername("tengxt");
        sysLog.setTime(time.toString());
        sysLog.setCreateTime(new Date());
        // 保存系统日志
        try {
            sysLogService.saveSysLog(sysLog);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 测试

```java
import com.tengxt.springbootaoplog.config.Log;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SysLogController {

    @GetMapping("/one")
    @Log("执行方法一")
    public void methodOne(String name){}

    @GetMapping("/two")
    @Log("执行方法二")
    public void methodTwo() throws InterruptedException{
        Thread.sleep(2000);
    }

    @GetMapping("/three")
    @Log("执行方法三")
    public void methodThree(String name, String age){}
}
```

启动项目，在数据库中能查看到数据就行了。

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-aop-log)












