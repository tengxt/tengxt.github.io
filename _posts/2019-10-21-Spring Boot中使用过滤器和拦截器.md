---
layout:     post
title:      "Spring Boot中使用过滤器和拦截器"
subtitle:   "Spring Boot 2.1.6"
date:       2019-10-21 19:27:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Filter
---

> 环境    Java 8   Spring Boot  2.1.6.RELEASE


过滤器（Filter）和拦截器（Interceptor）是Web项目中常用的两个功能，本文将简单介绍在Spring Boot中使用过滤器和拦截器来计算Controller中方法的执行时长，并且简单对比两者的区别。

现有如下Controller：

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @GetMapping("/user/{id:\\d+}")
    public void get(@PathVariable String id){
        System.out.println(id);
    }
}
```

下面通过配置过滤器和拦截器来实现对`get`方法执行时间计算的功能。

### 过滤器

定义一个`TimeFilter`类，实现`javax.servlet.Filter`。

```java
import javax.servlet.*;
import java.io.IOException;
import java.util.Date;

public class TimeFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器初始化");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("开始执行过滤器");
        long start = new Date().getTime();
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("[过滤器]耗时："+ (new Date().getTime() - start));
        System.out.println("结束执行过滤器");
    }
}
```

要使该过滤器在Spring Boot中生效，还需要一些配置。这里主要有两种配置方式。

#### 配置方式一

可通过在`TimeFilter`上加上如下注解：

```java
import org.springframework.stereotype.Component;
import javax.servlet.annotation.WebFilter;

@Component
@WebFilter(urlPatterns = {"/*"})
public class TimeFilter implements Filter {
   //...
}
```

`@Component`注解让`TimeFilter`成为Spring上下文中的一个Bean，`@WebFilter`注解的`urlPatterns`属性配置了哪些请求可以进入该过滤器，`/*`表示所有请求。

启动项目时可以看到控制台输出了`过滤器初始化`，启动后访问<http://localhost:8080/user/1>，控制台输出如下。

```powershell
过滤器初始化
开始执行过滤器
1
[过滤器]耗时：64
结束执行过滤器
```

#### 配置方式二

除了在过滤器类上加注解外，我们也可以通过`FilterRegistrationBean`来注册过滤器。

定义一个`WebConfig`类，加上`@Configuration`注解表明其为配置类，然后通过`FilterRegistrationBean`来注册过滤器。

```java
import com.tengxt.springbootfilterinterceptor.filter.TimeFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.ArrayList;
import java.util.List;

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        TimeFilter timeFilter = new TimeFilter();
        filterRegistrationBean.setFilter(timeFilter);

        List<String> urlList = new ArrayList<>();
        urlList.add("/*");

        filterRegistrationBean.setUrlPatterns(urlList);
        return filterRegistrationBean;
    }
}
```

`FilterRegistrationBean`除了注册过滤器`TimeFilter`外还通过`setUrlPatterns`方法配置了URL匹配规则。重启项目访问<http://localhost:8080/user/1>，我们可以看到和上面一样的效果。

> 通过过滤器只可以获取到`servletRequest`对象，所以并不能获取到方法的名称，所属类，参数等额外的信息。

### 拦截器

定义一个`TimeInterceptor`类，实现`org.springframework.web.servlet.HandlerInterceptor`接口。

```java
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;

public class TimeInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("处理拦截之前");
        request.setAttribute("startTime", new Date().getTime());
        System.out.println(((HandlerMethod)handler).getBean().getClass().getName());
        System.out.println(((HandlerMethod)handler).getMethod().getName());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("开始处理拦截");
        Long startTime = (Long) request.getAttribute("startTime");
        System.out.println("[拦截器]耗时：" + (new Date().getTime() - startTime));
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("处理拦截之后");
        Long startTime = (Long) request.getAttribute("startTime");
        System.out.println("[拦截器]耗时：" + (new Date().getTime() - startTime));
        System.out.println("异常信息："+ex);
    }
}
```

`TimeInterceptor`实现了`HandlerInterceptor`接口的三个方法。`preHandle`方法在处理拦截之前执行，`postHandle`只有当被拦截的方法没有抛出异常成功时才会处理，`afterCompletion`方法无论被拦截的方法抛出异常与否都会执行。

通过这三个方法的参数可以看到，相较于过滤器，拦截器多了Object和Exception对象，所以可以获取的信息比过滤器要多的多。但过滤器仍无法获取到方法的参数等信息，我们可以通过切面编程来实现这个目的，可参考[Spring Boot AOP记录用户操作日志](https://tengxt.gitee.io/2019/08/08/Spring-Boot-AOP%E8%AE%B0%E5%BD%95%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C%E6%97%A5%E5%BF%97/)。

要使拦截器在Spring Boot中生效，还需要如下两步配置：

1.在拦截器类（`TimeInterceptor`）上加入`@Component`注解；

2.在`WebConfig`中通过`InterceptorRegistry`注册过滤器。

```java
import org.springframework.beans.factory.annotation.Autowired;
import com.tengxt.springbootfilterinterceptor.interceptor.TimeInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

@Configuration
public class WebConfig  extends WebMvcConfigurationSupport {

    @Autowired
    private TimeInterceptor timeInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        //super.addInterceptors(registry);
        registry.addInterceptor(timeInterceptor);
    }
}
```

启动项目，访问<http://localhost:8080/user/1>，控制台输出如下。

```powershell
处理拦截之前
com.tengxt.springbootfilterinterceptor.controller.UserController
get
1
开始处理拦截
处理[拦截器]耗时：39
处理拦截之后
[拦截器]耗时：39
异常信息：null
```

从输出中我们可以了解到三个方法的执行顺序，并且三个方法都被执行了。

### 执行时机对比

我们将过滤器和拦截器都配置上，然后启动项目访问<http://localhost:8080/user/1>。

```powershell
过滤器初始化
开始执行过滤器
处理拦截之前
com.tengxt.springbootfilterinterceptor.controller.UserController
get
1
开始处理拦截
处理[拦截器]耗时：56
处理拦截之后
[拦截器]耗时：56
异常信息：null
[过滤器]耗时：76
结束执行过滤器
```

可看到过滤器要先于拦截器执行，晚于拦截器结束。下图很好的描述了它们的执行时间区别：

![filter.jpg](..\..\..\..\..\img\08\filter.jpg)


> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-filter-interceptor)













