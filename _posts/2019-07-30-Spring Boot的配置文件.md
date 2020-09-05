---
layout:     post
title:      "Spring Boot的配置文件"
subtitle:   "Spring Boot 2.1.6"
date:       2019-07-30 23:07:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
---


在 Spring Boot 中，配置文件有两种不同的格式，一个是 `properties` ，另一个是 `yaml` 。

## properties 配置

虽然 `properties` 文件比较常见，但是相对于` properties `而言，`yaml` 更加简洁明了，而且使用的场景也更多，很多开源项目都是使用 yaml 进行配置（例如 Hexo）。除了简洁，yaml 还有另外一个特点，就是 yaml 中的数据是有序的，`properties` 中的数据是无序的，在一些需要路径匹配的配置中，顺序就显得尤为重要（例如我们在` Spring Cloud Zuul `中的配置），此时我们一般采用 yaml。

### 位置问题

首先，当我们创建一个 Spring Boot 工程时，默认 resources 目录下就有一个 `application.properties` 文件，可以在 `application.properties` 文件中进行项目配置，但是这个文件并非唯一的配置文件，在 Spring Boot 中，一共有 4 个地方可以存放 application.properties 文件。

1. 当前项目根目录下的 config 目录下
2. 当前项目的根目录下
3. resources 目录下的 config 目录下
4. resources 目录下

![](..\..\..\..\img\02\springboot-application.properties.jpg)

这四个位置是默认位置，即 Spring Boot 启动，默认会从这四个位置按顺序去查找相关属性并加载。但是，这也不是绝对的，我们也可以在项目启动时自定义配置文件位置。

例如，现在在 `resources` 目录下创建一个 `tengxt `目录，目录中存放一个 `application.properties` 文件，那么正常情况下，当我们启动 Spring Boot 项目时，这个配置文件是不会被自动加载的。

```properties
spring.config.location=classpath:/tengxt/
```

我们可以通过 `spring.config.location` 属性来手动的指定配置文件位置，指定完成后，系统就会自动去指定目录下查找 `application.properties `文件。

![](..\..\..\..\img\02\properties-20190730220304.jpg)

此时启动项目，就会发现，项目以 `classpath:/tengxt/application.propertie` 配置文件启动。

这是在开发工具中配置了启动位置，如果项目已经打包成 jar ，在启动命令中加入位置参数即可：

```powershell
java -jar properties-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/tengxt/
```

### 文件名问题

对于 `application.properties` 而言，它不一定非要叫 `application` ，但是项目默认是去加载名为 `application` 的配置文件，如果我们的配置文件不叫 `application `，也是可以的，但是，需要明确指定配置文件的文件名。

方式和指定路径一致，只不过此时的 key 是 `spring.config.name` 。

首先我们在 `resources` 目录下创建一个 `app.properties` 文件，然后在 IDEA 中指定配置文件的文件名。

```powershell
spring.config.name=app
```

指定完配置文件名之后，再次启动项目，此时系统会自动去默认的四个位置下面分别查找名为 `app.properties` 的配置文件。当然，允许自定义文件名的配置文件不放在四个默认位置，而是放在自定义目录下，此时就需要明确指定 `spring.config.location` 。

配置文件位置和文件名称可以同时自定义。



### 普通的属性注入

由于 Spring Boot 源自 Spring ，所以 Spring 中存在的属性注入，在 Spring Boot 中一样也存在。由于 Spring Boot 中，默认会自动加载 `application.properties` 文件，所以简单的属性注入可以直接在这个配置文件中写。例如，定义一个 Book 类：

```java
public class Book {
    private Long id;
    private String name;
    private String author;
    //省略 getter/setter
}
```

然后，在 application.properties 文件中定义属性：

```properties
book.id=1
book.name=三国演义
book.author=罗贯中
```

按照传统的方式（Spring中的方式），可以直接通过 @Value 注解将这些属性注入到 Book 对象中

```java

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class Book {
    @Value("${book.id}")
    private Long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;
    //省略 getter/setter
}
```

**注意**

Book 对象本身也要交给 Spring 容器去管理，如果 Book 没有交给 Spring 容器，那么 Book 中的属性也无法从 Spring 容器中获取到值。

配置完成后，在 Controller 或者单元测试中注入 Book 对象，启动项目，就可以看到属性已经注入到对象中了。

```java
import com.tengxt.propertiesspringboot.entity.Book;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BookController {

    @Autowired
    private Book book;

    @GetMapping("/bookAll")
    public String books(){
        return book.getId() + book.getName() + book.getAuthor();
    }
}
```

### 类型安全的属性注入

Spring Boot 引入了类型安全的属性注入，如果采用 Spring 中的配置方式，当配置的属性非常多的时候，工作量就很大了，而且容易出错。

使用类型安全的属性注入，可以有效的解决这个问题。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "book")
public class Book {
    private Long id;
    private String name;
    private String author;
    //省略 getter/setter
}
```

这里，主要是引入 `@ConfigurationProperties(prefix = “book”) `注解，并且配置了属性的前缀，此时会自动将 Spring 容器中对应的数据注入到对象对应的属性中，就不用通过 `@Value` 注解挨个注入了，减少工作量并且避免出错。

## yaml 配置

首先`application.yaml`在Spring Boot中可以写在四个不同的位置，分别是如下位置：

1. 项目根目录下的config目录中
2. 项目根目录下
3. classpath下的config目录中
4. classpath目录下

四个位置中的`application.yaml`文件的优先级按照上面列出的顺序依次降低。即如果有同一个属性在四个文件中都出现了，以优先级高的为准。

那么`application.yaml`是不是必须叫`application.yaml`这个名字呢？当然不是必须的。开发者可以自己定义`yaml`名字，自己定义的话，需要在项目启动时指定配置文件的名字，像下面这样

```powershell
spring.config.name=app
```

当然这是在IntelliJ IDEA中直接配置的，如果项目已经打成jar包了，则在项目启动时加入如下参数：

```
java -jar myproject.jar --spring.config.name=app
```

这样配置之后，在项目启动时，就会按照上面所说的四个位置按顺序去查找一个名为`app.yaml`的文件。当然这四个位置也不是一成不变的，也可以自己定义，有两种方式，一个是使用`spring.config.location`属性，另一个则是使用`spring.config.additional-location`这个属性，在第一个属性中，表示自己重新定义配置文件的位置，项目启动时就按照定义的位置去查找配置文件，这种定义方式会覆盖掉默认的四个位置，也可以使用第二种方式，第二种方式则表示在四个位置的基础上，再添加几个位置，新添加的位置的优先级大于原本的位置。

配置方式如下：

```powershell
spring.config.name=app;spring.config.location=classpath:/myconfig/
```

**这里要注意，配置文件位置时，值一定要以斜杠（/）结尾。**

### 数组注入

```yaml
my:
  servers:
    dev.example.com
    another.example.com
```

这段数据可以绑定到一个带Bean的数组中

```java

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.ArrayList;
import java.util.List;

@Component
@ConfigurationProperties(prefix = "my")
public class Config {
    private List<String> servers = new ArrayList<>();

    public List<String> getServers(){
        return this.servers;
    }
}
```

项目启动后，配置中的数组会自动存储到servers集合中。当然，yaml不仅可以存储这种简单数据，也可以在集合中存储对象。

## 优缺点

不同于`properties`文件的无序，`yaml`配置是有序的，这一点在有些配置中是非常有用的，例如在`Spring Cloud Zuul`的配置中，当我们配置代理规则时，顺序就显得尤为重要了。当然yaml配置也不是万能的，例如：`yaml`配置目前不支持`@PropertySource`注解。

> [source code](https://github.com/tengxt/springboot-learn/tree/master/properties-spring-boot)









