---
layout:     post
title:      "Spring Boot 多数据源配置之MyBatis"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-07 20:55:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - MyBatis
    - DataSource
---

首先创建 MyBatis 项目，可以参考 [Spring Boot 中使用MyBatis](https://tengxt.gitee.io/2019/08/01/Spring-Boot%E4%B8%AD%E4%BD%BF%E7%94%A8-MyBatis/)

项目创建完成后，添加Druid依赖；和`JdbcTemplate`一样。`pom.xml`文件的完整依赖如下

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
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

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

#### 多数据源配置

在Spring Boot的配置文件`application.yml`中配置`MyBatis`多数据源和 [Spring Boot 多数据源配置之JdbcTemplate](https://tengxt.gitee.io/2019/08/01/Spring-Boot-%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE%E4%B9%8BJdbcTemplate/) 一致。

然后再提供两个`DataSource`如下：

在`com\tengxt\springbootmybatismultidatasource\config\`目录下创建`DataSourceConfig`数据源的配置文件

```java
import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @ConfigurationProperties(prefix  = "spring.datasource.druid.one")
    @Bean(name = "dsOne")
    DataSource dsOne(){
        return DruidDataSourceBuilder.create().build();
    }

    @ConfigurationProperties(prefix  = "spring.datasource.druid.two")
    @Bean(name = "dsTwo")
    DataSource dsTwo(){
        return DruidDataSourceBuilder.create().build();
    }
}
```

#### MyBatis配置

接下来则是`MyBatis`的配置，不同于`JdbcTemplate`，MyBatis的配置要稍微麻烦一些，因为要提供两个Bean，因此这里两个数据源我将在两个类中分开来配置，首先来看第一个数据源的配置

在`com\tengxt\springbootmybatismultidatasource\config\`目录下创建`MyBatisConfigOne`配置文件

```java
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = MyBatisConfigOne.PACKAGE, sqlSessionFactoryRef = "sqlSessionFactoryOne")
public class MyBatisConfigOne {

    // mapper 接口扫描路径
    static final String PACKAGE = "com.tengxt.springbootmybatismultidatasource.mapper";

    // mapper xml文件扫描路径
    static final String  MAPPER_LOCATION = "classpath:mapper/UserOneMapper.xml";


    @Resource(name = "dsOne")
    DataSource dsOne;

    @Bean(name = "sqlSessionFactoryOne")
    SqlSessionFactory sqlSessionFactoryOne(){
        SqlSessionFactory sqlSessionFactory = null;

        try {
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
            sqlSessionFactoryBean.setDataSource(dsOne);
            //如果不使用xml的方式配置mapper，则可以省去下面这行mapper location的配置。
            sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                    .getResources(MyBatisConfigOne.MAPPER_LOCATION));
            sqlSessionFactory = sqlSessionFactoryBean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return sqlSessionFactory;
    }

    @Bean
    SqlSessionTemplate sqlSessionTemplateOne() {
        return new SqlSessionTemplate(sqlSessionFactoryOne());
    }
}
```

创建`MyBatisConfigOne`类，首先指明该类是一个配置类（`@Configuration`），配置类中要扫描的包是`com.tengxt.springbootmybatismultidatasource.mapper`，即该包下的接口将操作`dsOne`中的数据，对应的`SqlSessionFactory`和`SqlSessionTemplate`分别是`sqlSessionFactoryOne`和`sqlSessionTemplateOne`，`SqlSessionFactory`根据`dsOne`创建，然后再根据创建好的`SqlSessionFactory`创建一个`SqlSessionTemplate`。

这里配置完成后，按照这个配置，再来配置第二个数据源就行了。

#### mapper创建

在`com\tengxt\springbootmybatismultidatasource\mapper\`目录下创建`UserOneMapper`接口文件

```java
import com.tengxt.springbootmybatismultidatasource.entity.User;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;

@Mapper
public interface UserOneMapper {
    List<User> queryUserOneAll();
}
```

在`resources\mapper\`目录下创建`UserOneMapper.xml`文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tengxt.springbootmybatismultidatasource.mapper.UserOneMapper">
    <resultMap id="BaseResultMap" type="com.tengxt.springbootmybatismultidatasource.entity.User">
        <id column="id" property="id" jdbcType="INTEGER"></id>
        <result column="name" property="name" jdbcType="VARCHAR"></result>
        <result column="sex" property="sex" jdbcType="BIT"></result>
    </resultMap>

    <select id="queryUserOneAll" resultMap="BaseResultMap">
        SELECT * FROM user
    </select>
</mapper>
```

`Service，Controller`以及测试数据参考[Spring Boot 多数据源配置之JdbcTemplate](https://tengxt.gitee.io/2019/08/01/Spring-Boot-%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE%E4%B9%8BJdbcTemplate/) 。

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-mybatis-multidatasource)












