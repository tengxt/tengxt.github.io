---
layout:     post
title:      "Spring Boot中使用 MyBatis"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-01 19:53:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - MyBatis
---

整合MyBatis之前，先搭建一个基本的Spring Boot项目[创建Spring-Boot项目](https://tengxt.gitee.io/2019/07/31/%E5%88%9B%E5%BB%BASpring-Boot%E9%A1%B9%E7%9B%AE/)。然后引入`mybatis-spring-boot-starter`和数据库（MySQL）连接驱动。

### `pom.xml`文件引入

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

不同版本的`Spring Boot`和`MyBatis`版本对应不一样，具体可查看[MyBatis官方文档](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)。

### Druid数据源

Druid是一个关系型数据库连接池，是阿里巴巴的一个开源项目；[Druid地址](https://github.com/alibaba/druid)。Druid不但提供连接池的功能，还提供监控功能，可以实时查看数据库连接池和SQL查询的工作情况。

#### 配置Druid依赖

Druid为`Spring Boot`项目提供了对应的starter

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

#### Druid数据源配置

使用Druid连接池，需要在`application.yml`下配置

```yaml
spring:
  datasource:
    druid:
      # 数据库访问配置, 使用druid数据源
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/mybatis_db?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
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

上述配置不但配置了Druid作为连接池，而且还开启了Druid的监控功能。 其他配置可参考[官方wiki](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

启动项目访问<http://localhost:8080/druid>

![](..\..\..\..\img\03\mybatis-springboot20190731103040.jpg)

输入账号密码即可看到Druid监控后台：

![](..\..\..\..\img\03\druid-springboot20190731103430.jpg)

关于Druid的更多说明，可查看[官方wiki文档](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

#### 创建对应实体

```
import java.io.Serializable;
public class User implements Serializable {
    private static final long serialVersionUID = -4367019591191910909L;
   	private Integer id;
	private String name;
	private byte sex;
	// 省略get/set	
}
```

##### Intellij idea用快捷键自动生成序列化id

类继承了`Serializable`接口之后，使用`alt+enter`快捷键自动创建序列化id 

进入`setting==>inspections==>serialization issues==>serializable class without ‘serialVersionUID’ `

![](..\..\..\..\img\03\mybatis-springboot20190731150514.jpg)

#### 创建一个包含基本`CRUD`的`UserMapper`

```java
public interface UserMapper {
    int save(User user);
    int update(User user);
    int deleteById(Integer id);
    User queryById(Integer id);
}
```

`UserMapper`的实现可以基于xml也可以基于注解。

##### 使用注解方式

```java
import com.tengxt.springbootmybatis.entity.User;
import org.apache.ibatis.annotations.*;
import org.springframework.stereotype.Component;

@Component
@Mapper
public interface UserMapper {
    @Insert("INSERT INTO user(name,sex) VALUES (#{name}, #{sex})")
    int save(User user);
    @Update("UPDATE user SET name=#{name},sex=#{sex} WHERE id=#{id}")
    int update(User user);
    @Delete("DELETE FROM user WHERE id=#{id}")
    int deleteById(Integer id);
    @Select("SELECT id,name,sex FROM user WHERE id=#{id}")
    @Results(id="User",value = {
            @Result(property = "id", column = "id", javaType = Integer.class),
            @Result(property = "name", column = "name", javaType = String.class),
            @Result(property = "sex", column = "sex", javaType = Byte.class)
    })
    User queryById(Integer id);
}
```

这里是通过全注解的方式来写SQL，不写XML文件，`@Select、@Insert、@Update以及@Delete`四个注解分别对应XML中的`select、insert、update以及delete`标签，`@Results`注解类似于XML中的`ResultMap`映射文件，另外使用`@SelectKey`注解可以实现主键回填的功能，即当数据插入成功后，插入成功的数据id会赋值到user对象的id属性上，具体可参考[MyBatis官方文档](www.mybatis.org/mybatis-3/zh/java-api.html)。

`UserMapper`创建好之后，还要配置`mapper`扫描，有两种方式，一种是直接在`UserMapper`上面添加`@Mapper`注解，这种方式有一个弊端就是所有的`Mapper`都要手动添加，要是落下一个就会报错，还有一个一劳永逸的办法就是直接在启动类上添加`Mapper`扫描，如下：

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan(basePackages = "com.tengxt.springbootmybatis.mapper")
public class SpringBootMybatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootMybatisApplication.class, args);
    }

}
```

##### 使用xml方式

使用xml方式需要在`application.yml`中进行一些额外的配置

```yaml
mybatis:
  # mapper xml实现扫描路径
  mapper-locations: classpath:mapper/*.xml
```

创建UserMapper.xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 封装返回结果 -->
<mapper namespace="com.tengxt.springbootmybatis.mapper.UserMapper">
    <resultMap id="BaseResultMap" type="com.tengxt.springbootmybatis.entity.User">
        <id column="id" property="id" jdbcType="INTEGER"></id>
        <result column="name" property="name" jdbcType="VARCHAR"></result>
        <result column="sex" property="sex" jdbcType="BYTE"></result>
    </resultMap>

    <!--useGeneratedKeys="true" keyProperty="id" 插入数据返回主键值-->
    <insert id="save" useGeneratedKeys="true" keyProperty="id" parameterType="com.tengxt.springbootmybatis.entity.User">
        INSERT INTO user(name,sex) VALUES (#{name}, #{sex})
    </insert>

    <update id="update" useGeneratedKeys="true" keyProperty="id" parameterType="com.tengxt.springbootmybatis.entity.User">
        UPDATE user SET name=#{name},sex=#{sex} WHERE id=#{id}
    </update>

    <delete id="deleteById" parameterType="java.lang.Integer">
        DELETE FROM user WHERE id=#{id}
    </delete>

    <select id="queryById" resultMap="BaseResultMap" parameterType="java.lang.Integer">
        SELECT id,name,sex FROM user WHERE id=#{id}
    </select>
</mapper>
```

#### 编写测试文件

`UserService`文件

```java
import com.tengxt.springbootmybatis.entity.User;

public interface UserService {
    int save(User user);
    int update(User user);
    int deleteById(Integer id);
    User queryById(Integer id);
}
```

`UserController`文件

```java
import com.tengxt.springbootmybatis.entity.User;
import com.tengxt.springbootmybatis.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/queryById/{id}")
    public User queryUserById(@PathVariable Integer id){
        return userService.queryById(id);
    }
}
```

> [source code](<https://github.com/tengxt/springboot-learn/tree/master/spring-boot-mybatis>)








