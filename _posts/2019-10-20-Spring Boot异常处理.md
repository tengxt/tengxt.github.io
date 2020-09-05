---
layout:     post
title:      "Spring Boot异常处理"
subtitle:   "Spring Boot 2.1.6"
date:       2019-10-20 21:16:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Exception
---

> 环境    Java 8   Spring Boot  2.1.6.RELEASE


Spring Boot对异常的处理有一套默认的机制：当应用中产生异常时，Spring Boot根据发送请求头中的`accept`是否包含`text/html`来分别返回不同的响应信息。当从浏览器地址栏中访问应用接口时，请求头中的`accept`便会包含`text/html`信息，产生异常时，Spring Boot通过`org.springframework.web.servlet.ModelAndView`对象来装载异常信息，并以HTML的格式返回；而当从客户端访问应用接口产生异常时（客户端访问时，请求头中的`accept`不包含`text/html`），Spring Boot则以JSON的格式返回异常信息。

### 默认异常处理机制

假设应用中有如下一个Controller

```java
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ExceptionDemoController {
    @RequestMapping("user/{id:\\d+}")
    public void get(@PathVariable String id){
        throw new RuntimeException("user not exist");
    }
}
```

在代码中我们主动的抛出了一个`RuntimeException`，使用浏览器访问<http://localhost:8080/user/1>。

页面返回了一些异常描述，并且请求头的`accpet`包含了`text/html`片段。

接着使用模拟发送REST请求的Chrome插件[Restlet Client](https://restlet.com/modules/client/)发送<http://localhost:8080/user/1>。

![exception-500.jpg](..\..\..\..\..\img\07\exception-500.jpg)

可以看到请求头的`accept`值为`*/*`，并且返回一段JSON格式的信息。

查看Spring Boot的`BasicErrorController`类便可看到这一默认机制的具体实现：

![exception-code.jpg](..\..\..\..\..\img\07\exception-code.jpg)

可看到`errorHtml`和`error`方法的请求地址和方法是一样的，唯一的区别就是`errorHtml`通过`produces = {"text/html"}`判断请求头的`accpet`属性中是否包含`text/html`，如果包含，便走该方法。

### 自定义html异常页面

我们可以通过在`src/main/resources/resources/error`路径下定义友好的异常页面，比如定义一个500.html页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>500</title>
</head>
<body>
    系统内部异常
</body>
</html>
```

自定义页面可选择放置的位置和执行的顺序。

![exception-error.jpg](..\..\..\..\..\img\07\exception-error.jpg)



然后再次通过浏览器访问:<http://localhost:8080/user/1>；页面结果：**系统内部异常**。

同样的，我们也可以定义404.html等常见的HTTP状态码对应的异常页面。

通过自定义html异常页面并不会影响客户端发送请求异常返回的结果。

### 自定义异常处理

除了可以通过自定义html异常页面来改变浏览器访问接口时产生的异常信息，我们也可以自定义异常处理来改表默认的客户端访问接口产生的异常信息。

我们手动定义一个`UserNotExistException`，继承`RuntimeException`。

```java
package com.tengxt.springbootexception.exception;

public class UserNotExistException extends RuntimeException {


    private static final long serialVersionUID = -2780674644150185417L;
    private String id;

    public UserNotExistException(String id){
        super("user not exist");
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}
```

然后定义一个Controller异常处理类`ControllerExceptionHandler`：

```java
import com.tengxt.springbootexception.exception.UserNotExistException;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class UserExceptionsHandler {

    @ExceptionHandler(UserNotExistException.class)
    @ResponseBody
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public Map<String, Object> handleUserNotExistsException(UserNotExistException e) {
        Map<String, Object> map = new HashMap<>();
        map.put("id", e.getId());
        map.put("message", e.getMessage());
        return map;
    }
}
```

其中注解`@ExceptionHandler`指定了要处理的异常类型，注解`@ResponseStatus`指定异常处理方法返回的HTTP状态码为`HttpStatus.INTERNAL_SERVER_ERROR`，即500。`org.springframework.http.HttpStatus`是一个spring自带的枚举类型，封装了常见的HTTP状态码及描述。

编写完自定义异常处理逻辑后，我们将UserController中的方法抛出的异常改为`UserNotExistException`

```java
@RequestMapping("user/{id:\\d+}")
public void get(@PathVariable String id){
    throw new UserNotExistException(id);
}
```


> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-exception)










