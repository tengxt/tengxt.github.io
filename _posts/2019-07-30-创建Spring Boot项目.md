---
layout:     post
title:      "创建 Spring Boot 项目"
subtitle:   "Spring Boot 2.1.6"
date:       2019-07-30 20:41:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
---

## Spring Boot 介绍

我们刚开始学习 `JavaWeb` 的时候，使用 `Servlet/JSP` 做开发，一个接口搞一个` Servlet` ，后来我们通过隐藏域或者反射等方式，可以减少 `Servlet` 的创建，但是依然不方便。再后来，我们引入` Struts2/SpringMVC` 这一类的框架，来简化我们的开发 ，和 `Servlet/JSP` 相比，引入框架之后，生产力确实提高了不少，但是久而久之，又发现了新的问题，即配置繁琐易出错，要做一个新项目，先搭建环境，环境搭建来搭建去，就是那几行配置，不同的项目，可能就是包不同，其他大部分的配置都是一样的，`Java` 总是被人诟病配置繁琐代码量大。因此`Spring Boot` 应运而生，`Spring Boot` 的特点：

1. 为所有基于 Spring 的 Java 开发提供方便快捷的入门体验。
2. 开箱即用，有自己自定义的配置就是用自己的，没有就使用官方提供的（默认的）。
3. 提供了一系列通用的非功能性的功能，例如嵌入式容器、安全管理、健康检测等。
4. 不需要XML配置。

`Spring Boot` 的出现让 Java 开发又回归简单，因为解决了开发中的痛点，因此这个技术得到了非常广泛的使用，`Spring Boot`在面试中基本上就是必问，现在流行的 `Spring Cloud` 微服务也是基于 `Spring Boot`，因此，所有的 Java 工程师都有必要掌握好 `Spring Boot`。

## 系统要求

`Spring Boot` 目前（2019.7.30）最新稳定版本是 2.1.6，要求至少` JDK8`集成的 Spring 版本是 5.1.6 ，构建工具版本要求如下：

| Build Tool | Version |
| ---------- | ------- |
| Maven      | 3.3+    |
| Gradle     | 4.4+    |

内置的容器版本分别如下：

| Name         | Version |
| ------------ | ------- |
| Tomcat 9.0   | 4.0     |
| Jetty 9.4    | 3.1     |
| Undertow 2.0 | 4.0     |

## 三种创建方式

其实 `Spring Boot` 工程本质上就是一个 `Maven` 工程，从这个角度出发，有三种创建项目的方式。

### 1. 在线创建

这是官方提供的创建方式，实际上，如果我们使用开发工具去创建` Spring Boot` 项目（即第二种方案），也是从这个网站上创建的，只不过这个过程开发工具帮助我们完成了，我们只需要在开发工具中进行简单的配置即可。

首先打开 `https://start.spring.io` 这个网站，如下：

![](../../../../img/01/start-SpringBoot20190730154302.jpg)



这里要配置的按顺序分别如下：

- 项目构建工具是 Maven  还是 Gradle ？有人用 Gradle 做 Java 后端项目，但是整体感觉 Gradle 在 Java 后端中使用的还是比较少，Gradle 在 Android 中使用较多，Java 后端，目前来看还是 Maven 为主，因此这里选择Maven 。
- 开发语言，这个当然是选择 Java 了。
- `Spring Boot` 版本，可以看到，目前最新的稳定版是` 2.1.6`。
- 既然是 Maven 工程，当然要有项目坐标，项目描述等信息了，另外这里还让输入了包名，因为创建成功后会自动创建启动类。
- Packing 表示项目要打包成 jar 包还是 war 包，`Spring Boot `的一大优势就是内嵌了` Servlet `容器，打成 jar 包后可以直接运行，所以这里建议打包成 jar 包，当然，开发者根据实际情况也可以选择 war 包。
- 然后选择构建的 JDK 版本。
- 最后是选择所需要的依赖，输入关键字如 web ，会有相关的提示，这里我就先加入 web 依赖。

所有的事情全部完成后，点击最下面的 `Generate Project` 按钮，或者点击 `Alt+Enter` 按键，此时会自动下载项目，将下载来的项目解压，然后用 IntelliJ IDEA 或者 Eclipse 打开即可进行开发。

### 2. 使用开发工具创建

上面的步骤太过于繁琐，那么也可以使用 IDE 来创建，需要注意的是，IntelliJ IDEA 只有 ultimate 版才有直接创建 `Spring Boot` 项目的功能，社区版是没有此项功能的。

#### **IntelliJ IDEA**

首先在创建项目时选择 Spring Initializr，如下图：

![](../../../../img/01/start-springboot20190730160408.jpg)

然后点击 Next ，填入 Maven 项目的基本信息，如下：

![](../../../../img/01/start-springboot20190730160833.jpg)

再接下来选择需要添加的依赖，如下图：

![](../../../../img/01/start-springboot20190730160948.jpg)

勾选完成后，点击 Next 完成项目的创建。

### 3. Maven 创建

上面提到的几种方式，实际上都借助了 `https://start.spring.io/` 这个网站，也可以直接使用 Maven 来创建项目。步骤如下：

首先创建一个普通的 Maven 项目，以 IntelliJ IDEA 为例，创建步骤如下：

![](../../../../img/01/start-springboot20190730161955.jpg)

注意这里不用选择项目骨架，直接点击 Next ，下一步中填入一个 Maven 项目的基本信息，如下图：

![](../../../../img/01/start-springboot20190730162155.jpg)

然后点击 Next 完成项目的创建。

创建完成后，在 `pom.xml` 文件中，添加如下依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

添加成功后，再在 java 目录下创建包，包中创建一个名为 App 的启动类。为了演示简单，不再新建控制器，直接在入口类中编写测试代码。如下：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@EnableAutoConfiguration  // 开启自动化配置。
@RestController			  // ==》 @Controller + @ResponseBody
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @GetMapping("/hello")
    public String hello(){
        return "Hello Spring Boot";
    }
}
```

然后执行这里的 `main` 方法就可以启动一个 `Spring Boot` 工程了。

在浏览器访问地址`http://localhost:8080/hello`可在页面显示`Hello Spring Boot` 说明配置成功。

![](../../../../img/01/start-springboot20190730163146.jpg)

##  Spring Boot 项目中的 parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

### 基本功能

当我们创建一个 `Spring Boot` 工程时，可以继承 `spring-boot-starter-parent` ，也可以不继承它，先来看 `parent` 的基本功能有哪些？

1. 定义了 Java 编译版本为 1.8 。
2. 使用 UTF-8 格式编码。
3. 继承自 `spring-boot-dependencies`，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对 `application.properties` 和 `application.yml` 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 `application-dev.properties` 和 `application-dev.yml`。

**请注意，由于application.properties和application.yml文件接受Spring样式占位符 $ {...} ，因此 Maven 过滤更改为使用 @ .. @ 占位符，当然开发者可以通过设置名为 resource.delimiter 的Maven 属性来覆盖 @ .. @ 占位符。**

### 源码分析

当我们创建一个 Spring Boot 项目后，我们可以在本地 Maven 仓库中看到看到这个具体的 parent 文件，以 2.1.6 这个版本为例，路径是 `xxx\org\springframework\boot\spring-boot-starter-parent\2.1.6.RELEASE\spring-boot-starter-parent-2.1.6.RELEASE.pom` ,打开这个文件，快速阅读文件源码，基本上就可以证实我们前面说的功能，如下图：

```xml
<?xml version="1.0" encoding="utf-8"?><project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath>spring-boot-dependencies</relativePath>
  </parent>
  <artifactId>spring-boot-starter-parent</artifactId>
  <packaging>pom</packaging>
  <name>Spring Boot Starter Parent</name>
  <description>Parent pom providing dependency and plugin management for applications
		built with Maven</description>
  <url>https://projects.spring.io/spring-boot/#/spring-boot-starter-parent</url>
  <properties>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <resource.delimiter>@</resource.delimiter>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.target>${java.version}</maven.compiler.target>
  </properties>
  <build>
    <resources>
      <resource>
        <filtering>true</filtering>
        <directory>${basedir}/src/main/resources</directory>
        <includes>
          <include>**/application*.yml</include>
          <include>**/application*.yaml</include>
          <include>**/application*.properties</include>
        </includes>
      </resource>
      <resource>
        <directory>${basedir}/src/main/resources</directory>
        <excludes>
          <exclude>**/application*.yml</exclude>
          <exclude>**/application*.yaml</exclude>
          <exclude>**/application*.properties</exclude>
        </excludes>
      </resource>
    </resources>
     // ....
```

我们可以看到，它继承 `spring-boot-dependencies` ，这里保存了基本的依赖信息，另外我们也可以看到项目的编码格式，JDK 的版本等信息，当然也有我们前面提到的数据过滤信息。最后，我们再根据它的 parent 中指定的 `spring-boot-dependencies` 位置，来看看`spring-boot-dependencies` 中的版本信息定义：

```xml
<properties>
    <activemq.version>5.15.9</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.75</appengine-sdk.version>
    <artemis.version>2.6.4</artemis.version>
    <aspectj.version>1.9.4</aspectj.version>
    <assertj.version>3.11.1</assertj.version>
    <atomikos.version>4.0.6</atomikos.version>
    <bitronix.version>2.1.4</bitronix.version>
    <build-helper-maven-plugin.version>3.0.0</build-helper-maven-plugin.version>
    <byte-buddy.version>1.9.13</byte-buddy.version>
    <caffeine.version>2.6.2</caffeine.version>
    <cassandra-driver.version>3.6.0</cassandra-driver.version>
    <classmate.version>1.4.0</classmate.version>
    <commons-codec.version>1.11</commons-codec.version>
    <commons-dbcp2.version>2.5.0</commons-dbcp2.version>
    <commons-lang3.version>3.8.1</commons-lang3.version>
    <commons-pool.version>1.6</commons-pool.version>
    <commons-pool2.version>2.6.2</commons-pool2.version>
    <couchbase-cache-client.version>2.1.0</couchbase-cache-client.version>
    <couchbase-client.version>2.7.7</couchbase-client.version>
    // ....
```

在这里，我们看到了版本的定义以及 `dependencyManagement `节点，明白了为啥 Spring Boot 项目中部分依赖不需要写版本号了。


> [source code](https://github.com/tengxt/springboot-learn/tree/master/start-spring-boot)








