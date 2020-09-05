---
layout:     post
title:      "Spring Boot Devtools热部署"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-22 22:27:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Devtools
---

> 环境：jdk 8  spring boot  2.1.7.RELEASE

在开发项目中，修改了Java代码或者配置文件的时候，必须手动重启项目才能生效。所谓的热部署就是在你修改了后端代码后不需要手动重启，工具会帮你快速的自动重启是修改生效。其深层原理是使用了两个`ClassLoader`，一个`Classloader`加载那些不会改变的类（第三方Jar包），另一个`ClassLoader`加载会更改的类，称为`restart ClassLoader`，这样在有代码更改的时候，原来的`restart ClassLoader` 被丢弃，重新创建一个`restart ClassLoader`，由于需要加载的类相比较少，所以实现了较快的重启时间。即devtools会监听classpath下的文件变动，并且会立即重启应用（发生在保存时机）

### 引入Devtools

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

devtools会监听`classpath`目录下的文件变动，并且会立即重启应用（发生在保存时机），因为其采用的虚拟机机制，该项重启是很快的。

需要修改`spring-boot-maven-plugin`插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!--fork:如果没有该项配置,整个devtools不会起作用-->
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 配置IDEA

1) 设置IDEA的自动编译

打开` Settings（Ctrl+Alt+S） –> Build,Execution,Deployment –>Compiler`，将 `Build project automatically`.打钩。

![](../../../../../img/05/springboot-devtools_20190822214250.jpg)

2）组合键 `Shift+Ctrl+Alt+/`，选择 `Registry` ，找到`compiler.automake.allow.when.app.running`，选中打勾。（或者CTRL + SHIFT + A --> 查找Registry）

![](../../../../../img/05/spring-boot_20190822221108.jpg)

### 浏览器设置

设置游览器禁用缓存：F12（或Ctrl+Shift+J或Ctrl+Shift+I），打开开发者工具 → NetWork → 勾选`Disable Cache`

![](../../../../../img/05/spring-boot-web-cache_20190822221300.jpg)

至此配置完成，修改文件后不用重启，刷新页面即可看到效果

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-devtools)










