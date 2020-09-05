---
layout:     post
title:      "Spring Boot 多数据源配置之JdbcTemplate"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-01 22:27:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - JdbcTemplate
    - DataSource
---

多数据源配置也算是一个常见的开发需求，在`Spring Boot`中，`JdbcTemplate、MyBatis以及Jpa`都可以配置多数据源。

先创建应用，可以参考[Spring Boot中使用 MyBatis](https://tengxt.gitee.io/2019/08/02/Spring-Boot%E4%B8%AD%E4%BD%BF%E7%94%A8-MyBatis/)

#### 引入依赖

创建成功之后，接下来手动添加Druid依赖，由于这里一会需要开发者自己配置`DataSoruce`，所以这里必须要使用`druid-spring-boot-starter`依赖，而不是传统的那个druid依赖，因为`druid-spring-boot-starter`依赖提供了`DruidDataSourceBuilder`类，这个可以用来构建一个`DataSource`实例，而传统的Druid则没有该类。完整的依赖如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

#### 多数据源配置

接着在`Spring Boot`配置文件`application.yml`中配置多数据源

```yaml
spring:
  datasource:
    druid:
      # 数据库访问配置, 使用druid数据源
      # 数据源1
      one:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/test01?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
        username: root
        password: 123
      # 数据源2
      two:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/test02?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
        username: root
        password: 123

      # 连接池配置
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 连接等待超时时间
      max-wait: 30000
      # 配置检测可以关闭的空闲连接间隔时间
      time-between-eviction-runs-millis: 60000
      # 配置连接在池中的最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: select '1' from dual
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-open-prepared-statements: 20
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙
      filters: stat,wall
      # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
      aop-patterns: com.tengxt.springbootmybatis.service.*

      # WebStatFilter配置
      web-stat-filter:
        enabled: true
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤的格式
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      # StatViewServlet配置
      stat-view-servlet:
        enabled: true
        # 访问路径为/druid时，跳转到StatViewServlet
        url-pattern: /druid/*
        # 是否能够重置数据
        reset-enable: false
        # 需要账号密码才能访问控制台
        login-username: druid
        login-password: druid
        # IP白名单
        # allow: 127.0.0.1
        #　IP黑名单（共同存在时，deny优先于allow）
        # deny: 192.168.1.218

      # 配置StatFilter
      filter:
        stat:
          log-slow-sql: true
```

这里通过one和two对数据源进行了区分，但是加了one和two之后，这里的配置就没法被`Spring Boot`自动加载了（因为前面的key变了），需要我们自己去加载`DataSource`了，此时，需要自己配置一个`DataSourceConfig`，用来提供两个`DataSource Bean`

```java
import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.druid.one")
    DataSource dsOne(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.druid.two")
    DataSource dsTwo(){
        return DruidDataSourceBuilder.create().build();
    }
	//   @Qualifier   按照名称查找
    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne")DataSource dsOne){
        return new JdbcTemplate(dsOne);
    }

    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo")DataSource dsTwo){
        return new JdbcTemplate(dsTwo);
    }
}
```

这里提供了两个Bean，其中`@ConfigurationProperties`是`Spring Boot`提供的类型安全的属性绑定，以第一个Bean为例，`@ConfigurationProperties(prefix = "spring.datasource.druid.one")`表示使用`spring.datasource.druid.one`前缀的数据库配置去创建一个`DataSource`，这样配置之后，我们就有了两个不同的`DataSource`，接下来再用这两个不同的`DataSource`去创建两个不同的`JdbcTemplate`。

#### 测试数据

创建两个数据库，并创建两张表；

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `sex` bit(1) DEFAULT NULL COMMENT '(0：女 1：男）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', '张三', true);
INSERT INTO `user` VALUES ('2', '李四', false);
INSERT INTO `user` VALUES ('3', '王五', true);
```

#### service层

和`DataSource`一样，Spring容器中的`JdbcTemplate`也是有两个，因此不能通过`byType`的方式注入进来，这里给大伙提供了两种注入思路，一种是使用`@Resource`注解，直接通过 byName 的方式注入进来，另外一种就是`@Autowired`注解加上`@Qualifier`注解，两者联合起来，实际上也是 byName 。将 `JdbcTemplate`注入进来之后，`jdbcTemplateOne`和`jdbcTemplateTwo`此时就代表操作不同的数据源，使用不同的 `JdbcTemplate`操作不同的数据源，实现了多数据源配置。

```java

import com.tengxt.springbootjdbctemplatemultidatasource.entity.User;
import com.tengxt.springbootjdbctemplatemultidatasource.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    @Qualifier("jdbcTemplateOne")
    private JdbcTemplate jdbcTemplateOne;

    @Resource(name = "jdbcTemplateTwo")
    private JdbcTemplate jdbcTemplateTwo;

    @Override
    public List<User> queryUser01All() {
        String sql = "SELECT * FROM user";
        List<User> userList = jdbcTemplateOne.query(sql, new BeanPropertyRowMapper<>(User.class));
        if(null != userList && userList.size() > 0){
            return userList;
        }
        return null;
    }

    @Override
    public List<User> queryUser02All() {
        String sql = "SELECT * FROM user";
        List<User> userList = jdbcTemplateOne.query(sql, new BeanPropertyRowMapper<>(User.class));
        if(null != userList && userList.size() > 0){
            return userList;
        }
        return null;
    }
}
```

#### Controller层

直接调用`service`层的数据返回给页面

```java
import com.tengxt.springbootjdbctemplatemultidatasource.entity.User;
import com.tengxt.springbootjdbctemplatemultidatasource.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("queryAllOne")
    public List<User> queryAllOne(){ return userService.queryUser01All(); }

    @GetMapping("/queryAllTwo")
    public List<User> queryAllTwo(){ return userService.queryUser02All(); }
}
```

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-jdbctemplate-multidatasource)










