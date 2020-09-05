---
layout:     post
title:      "Spring Boot 中使用 Thymeleaf "
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-08 19:14:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Thymeleaf
---

早期的 `Spring Boot` 中还支持使用 `Velocity` 作为页面模板，现在的 `Spring Boot` 中已经不支持 `Velocity` 了，页面模板主要支持 `Thymeleaf` 和 `Freemarker` ，当然，作为 Java 最基本的页面模板 `Jsp` ，`Spring Boot `也是支持的，只是使用比较麻烦。

### Thymeleaf 简介

Thymeleaf 是新一代 Java 模板引擎，它类似于 Velocity、FreeMarker 等传统 Java 模板引擎，但是与传统 Java 模板引擎不同的是，Thymeleaf 支持 HTML 原型。

它既可以让前端工程师在浏览器中直接打开查看样式，也可以让后端工程师结合真实数据查看显示效果，同时，SpringBoot 提供了 Thymeleaf 自动化配置解决方案，因此在 SpringBoot 中使用 Thymeleaf 非常方便。

事实上， `Thymeleaf` 除了展示基本的 HTML ，进行页面渲染之外，也可以作为一个 HTML 片段进行渲染，例如做邮件发送时，可以使用 `Thymeleaf` 作为邮件发送模板。另外，由于 Thymeleaf 模板后缀为 `.html`，可以直接在浏览器打开；因此，预览时非常方便。

#### 引入 pom.xml 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

当然，Thymeleaf 不仅仅能在 Spring Boot 中使用，也可以使用在其他地方，只不过 Spring Boot 针对 Thymeleaf 提供了一整套的自动化配置方案，这一套配置类的属性在 `org.springframework.boot.autoconfigure.thymeleaf.ThymeleafProperties` 中，部分源码如下：

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
    // 默认的编码格式
    private static final Charset DEFAULT_ENCODING; 
    // 视图解析器的前缀
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    // 视图解析器的后缀
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    private Charset encoding;
    private boolean cache;
    private Integer templateResolverOrder;
    private String[] viewNames;
    private String[] excludedViewNames;
    private boolean enableSpringElCompiler;
    private boolean renderHiddenMarkersBeforeCheckboxes;
    private boolean enabled;
    private final ThymeleafProperties.Servlet servlet;
    private final ThymeleafProperties.Reactive reactive;

    public ThymeleafProperties() {
        this.encoding = DEFAULT_ENCODING;
        this.cache = true;
        this.renderHiddenMarkersBeforeCheckboxes = false;
        this.enabled = true;
        this.servlet = new ThymeleafProperties.Servlet();
        this.reactive = new ThymeleafProperties.Reactive();
    }
    // ....
```

1. 首先通过 `@ConfigurationProperties` 注解，将 `application.properties` 前缀为 `spring.thymeleaf` 的配置和这个类中的属性绑定。
2. 前三个 `static` 变量定义了默认的编码格式、视图解析器的前缀、后缀等。
3. 从前三行配置中，可以看出来，`Thymeleaf` 模板的默认位置在 `resources/templates` 目录下，默认的后缀是 `html` 。
4. 这些配置，如果开发者不自己提供，则使用 默认的，如果自己提供，则在 `application.properties` 中以 `spring.thymeleaf` 开始相关的配置。
5. 一般开发中将`spring.thymeleaf.cache`设置为false，其他保持默认值即可。

Spring Boot 为 Thymeleaf 提供的自动化配置类，则是 `org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration` ，部分源码如下：

```java
@Configuration
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration {
    // ...
}
```

可以看到，在这个自动化配置类中，首先导入 `ThymeleafProperties` ，然后 `@ConditionalOnClass` 注解表示当当前系统中存在 `TemplateMode` 和 `SpringTemplateEngine` 类时，当前的自动化配置类才会生效，即只要项目中引入了 `Thymeleaf` 相关的依赖，这个配置就会生效。

这些默认的配置我们几乎不需要做任何更改就可以直接使用了。如果开发者有特殊需求，则可以在 `application.properties` 中配置以 `spring.thymeleaf` 开头的属性即可。

#### 创建Controller类

```java
import com.tengxt.springbootthymeleaf.entity.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.ArrayList;
import java.util.List;

@Controller
public class UserController {

    @GetMapping("/index")
    public String index(Model model){
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 5; i++){
            User user = new User();
            user.setId((long) i);
            user.setName("tengxt" + i);
            user.setAddress("火星" + i);
            users.add(user);
        }
        model.addAttribute("users", users);
        return "index";
    }
}
```

在 `UserController` 中返回逻辑视图名+数据，逻辑视图名为 `index` ，意思我们需要在`resources/templates` 目录下提供一个名为 `index.html` 的 `Thymeleaf` 模板文件

#### 创建 Thymeleaf模板文件

```html
<!DOCTYPE html>
<html lang="en">
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
<table>
    <tr>
        <th>id</th>
        <th>name</th>
        <th>address</th>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.id}"></td>
        <td th:text="${user.name}"></td>
        <td th:text="${user.address}"></td>
    </tr>
</table>
</body>
</html>
```

在 `Thymeleaf` 中，通过 `th:each` 指令来遍历一个集合，数据的展示通过 `th:text` 指令来实现，

注意： `index.html` 最上面要引入 `thymeleaf` 名称空间（`<html xmlns:th="http://www.thymeleaf.org">`）。

配置完成后，就可以启动项目了，访问 `/index` 接口，就能看到数据了。

另外，`Thymeleaf` 支持在 `js` 中直接获取 `Model` 中的数据。

```javascript
<script th:inline="javascript">
    var users = [[${users}]];
	// 在控制台输入数据
    console.log(users);
</script>
```

#### 手动渲染

可以手动渲染 Thymeleaf 模板，这个一般在邮件发送时候有用；[Spring Boot发送邮件](https://tengxt.gitee.io/2019/08/08/Spring-Boot-%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6(Thymeleaf)%E6%A8%A1%E6%9D%BF/)

关于 Thymeleaf 模板更多的用法，参考[Thymeleaf官网文档](https://www.thymeleaf.org/)

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-thymeleaf)
















