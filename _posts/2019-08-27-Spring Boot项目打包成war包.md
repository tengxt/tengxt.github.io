---
layout:     post
title:      "Spring Boot项目打包成war包"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-27 23:05:00
author:     "Xt"
header-img: "img/post-bg-war.jpg"
tags:
    - Spring Boot 2.1.6
    - War
---

> 环境    Java 8   Spring Boot  2.1.7.RELEASE

在`pom.xml`文件中，将打包方式改成`war`

```xml
<packaging>war</packaging>
```

然后添加Tomcat依赖配置，覆盖Spring Boot自带的Tomcat依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

在`<build></build>`标签内配置项目名（该配置类似于`server.context-path=tengxt`）

```xml
<build>
    //...
     <finalName>tengxt</finalName>
    //...
</build>
```

在启动类`ServletInitializer` 中继承（`extends`）类`SpringBootServletInitializer`

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class SpringBootWarApplication extends SpringBootServletInitializer {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootWarApplication.class, args);
	}

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return builder.sources(SpringBootWarApplication.class);
	}
}
```

准备完毕后，运行`mvn clean package`命令即可在target目录下查看生产war包。


> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-war)









