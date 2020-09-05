---
layout:     post
title:      "Spring Boot中使用 JdbcTemplate"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-01 22:27:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - JdbcTemplate
---

在Java领域，数据持久化有几个常见的方案，有 Spring 自带的 JdbcTemplate、MyBatis 和 JPA，在这些方案中，最简单的就是 Spring 自带的 JdbcTemplate 了，这个东西虽然没有 MyBatis 那么方便，但是比起最开始的 Jdbc 已经强了很多了，它没有 MyBatis 功能那么强大，当然也意味着它的使用比较简单。事实上，JdbcTemplate 算是最简单的数据持久化方案了。

Spring Boot开启 JdbcTemplate 很简单，只需要引入`spring-boot-starter-jdbc`依赖即可。JdbcTemplate 封装了许多SQL操作，具体可查阅[官方文档](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html)。

#### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

数据库驱动为MySQL，数据源采用Druid。可以参考[Spring-Boot中使用-MyBatis](https://tengxt.gitee.io/2019/08/02/Spring-Boot%E4%B8%AD%E4%BD%BF%E7%94%A8-MyBatis/)

#### 测试数据

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

#### 编写代码

在`UserServiceImpl`实现类里使用`JdbcTemplate`代码

```java
import com.tengxt.springbootjdbctemplate.service.UserService;
import com.tengxt.springbootjdbctemplate.entity.User;
import com.tengxt.springbootjdbctemplate.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.sql.Types;
import java.util.List;
import java.util.Map;

@Repository("userService")
public class UserServiceImpl implements UserService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public int save(User user) {
        String sql = "INSERT INTO user(name,sex) VALUES(?,?)";
        Object[] args = {user.getName(), user.getSex()};
        int[] argTypes = {Types.VARCHAR, Types.BIT};
        return this.jdbcTemplate.update(sql, args, argTypes);
    }

    @Override
    public int update(User user) {
        String sql = "UPDATE user SET name = ? , sex = ? WHERE id = ?";
        Object[] args = {user.getName(), user.getSex(), user.getId()};
        int[] argTypes = {Types.VARCHAR, Types.BIT, Types.INTEGER};
        return this.jdbcTemplate.update(sql, args, argTypes);
    }

    @Override
    public int deleteById(Integer id) {
        String sql = "DELETE FROM user WHERE id =?";
        Object[] args = {id};
        int[] argTypes = {Types.INTEGER};
        return this.jdbcTemplate.update(sql, args, argTypes);
    }

    /**
     * 查询所有
     * @return
     */
    @Override
    public List<Map<String, Object>> queryUsersListMap() {
        String sql = "SELECT id,name,sex FROM user";
        return this.jdbcTemplate.queryForList(sql);
    }

    /**
     * 根据id查询
     * @param id
     * @return
     */
    @Override
    public User queryUsersById(Integer id) {
        String sql = "SELECT id,name,sex FROM user WHERE id = ?";
        Object[] args = {id};
        int[] argTypes = {Types.INTEGER};
        List<User> userList = this.jdbcTemplate.query(sql, args, argTypes, new UserMapper());
        if(null != userList && userList.size() > 0){
            return userList.get(0);
        }
        return null;
    }
}
```

查询的时候需要提供一个`RowMapper`，就是需要自己手动映射，将数据库中的字段和对象的属性一一对应起来。

```java
import com.tengxt.springbootjdbctemplate.entity.User;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 返回表对应的实体对象
 */
public class UserMapper implements RowMapper<User> {
	@Override
    public User mapRow(ResultSet resultSet, int i) throws SQLException {
        User user = new User();
        user.setId(resultSet.getInt("id"));
        user.setName(resultSet.getString("name"));
        user.setSex(resultSet.getByte("sex"));
        return user;
    }
}
```

emmm 看起来有点麻烦，如果数据库中的字段和对象属性的名字一模一样的话，有一个简单的方案，如下：

```java
public User queryUsersById(Integer id) {
        String sql = "SELECT id,name,sex FROM user WHERE id = ?";
        Object[] args = {id};
        int[] argTypes = {Types.INTEGER};
        List<User> userList = this.jdbcTemplate.query(sql, args, argTypes, new BeanPropertyRowMapper<>(User.class));
        if(null != userList && userList.size() > 0){
            return userList.get(0);
        }
        return null;
    }
```

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-jdbctemplate)










