---
layout:     post
title:      "Spring Boot 中使用 Redis 缓存数据 "
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-14 22:09:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Redis
---
在程序中可以使用缓存的技术来节省对数据库的开销。Spring Boot对缓存提供了很好的支持，我们几乎不用做过多的配置即可使用各种缓存实现。这里主要介绍`Ehcache`和`Redis`缓存实现。

### 创建项目

搭建Spring Boot应用参考[Spring-Boot中使用-MyBatis](https://tengxt.gitee.io/2019/08/01/Spring-Boot%E4%B8%AD%E4%BD%BF%E7%94%A8-MyBatis/)，然后在配置文件中添加日志输出级别用以观察`SQL`的执行情况

```yaml
# 配置日志输出级别以观察SQL的执行情况：
logging:
  level:
    com:
      tengxt:
        springbootrediscache:
          mapper: debug
```

其中`com.tengxt.springbootrediscache.mapper`为MyBatis的Mapper包（`package`）路径。

然后编写如下测试方法

```java
import com.tengxt.springbootrediscache.entity.SysLog;
import com.tengxt.springbootrediscache.service.SysLogService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootRedisCacheApplicationTests {

    @Autowired
    private SysLogService sysLogService;

    @Test
    public void contextLoads() {
        SysLog sysLog1 = this.sysLogService.querySysLogById(1);
        System.out.println(sysLog1);

        SysLog sysLog2 = this.sysLogService.querySysLogById(1);
        System.out.println(sysLog2);
    }
}
```

右键运行

```java
c.t.s.m.SysLogMapper.querySysLogById     : ==>  Preparing: SELECT * FROM sys_log WHERE id = ? 
2019-08-13 18:05:51.512 DEBUG 19552 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : ==> Parameters: 1(Integer)
2019-08-13 18:05:51.552 DEBUG 19552 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : <==      Total: 1
SysLog{id=1, username='tengxt', operation='执行方法一', time='3', method='com.tengxt.springbootaoplog.controller.SysLogController.methodOne()', ip='127.0.0.1', createTime=Tue Aug 06 08:00:00 CST 2019, params=' name : null'}
2019-08-13 18:05:51.603 DEBUG 19552 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : ==>  Preparing: SELECT * FROM sys_log WHERE id = ? 
2019-08-13 18:05:51.604 DEBUG 19552 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : ==> Parameters: 1(Integer)
2019-08-13 18:05:51.626 DEBUG 19552 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : <==      Total: 1
SysLog{id=1, username='tengxt', operation='执行方法一', time='3', method='com.tengxt.springbootaoplog.controller.SysLogController.methodOne()', ip='127.0.0.1', createTime=Tue Aug 06 08:00:00 CST 2019, params=' name : null'}
2019-08-13 18:05:51.653  INFO 19552 --- [       Thread-2] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
Process finished with exit code 0
```

可发现查询都是进行的数据库查询

### 使用缓存

要开启Spring Boot的缓存功能，需要在`pom.xml`文件中引入`spring-boot-starter-cache`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

接着在Spring Boot入口类中加入`@EnableCaching`注解开启缓存功能

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching  //开启缓存
public class SpringBootRedisCacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootRedisCacheApplication.class, args);
    }
}
```

### 缓存注解

在`service`接口中添加缓存注解

#### @CacheConfig

这个注解在类上使用，用来描述该类中所有方法使用的缓存名称，当然也可以不使用该注解，直接在具体的缓存注解上配置名称，示例代码如下：

```java
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.stereotype.Service;

@Service("sysLogService")
@CacheConfig(cacheNames = "sysLog")
public class SysLogServiceImpl implements SysLogService {
    
}
```

#### @Cacheable

这个注解一般加在查询方法上，表示将一个方法的返回值缓存起来，默认情况下，缓存的key就是方法的参数，缓存的value就是方法的返回值。示例代码如下：

```java
import org.springframework.cache.annotation.Cacheable;

@Override
@Cacheable
public SysLog querySysLogById(Integer id) {
    return sysLogMapper.querySysLogById(id);
}
```

#### @CachePut

这个注解一般加在更新方法上，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的key上，示例代码如下：

```java
import org.springframework.cache.annotation.CachePut;

@Override
@CachePut(key = "#id")
public int update(String username, Integer id) {
    return sysLogMapper.update(username, id);
}
```

#### @CacheEvict

这个注解一般加在删除方法上，当数据库中的数据删除后，相关的缓存数据也要自动清除，该注解在使用的时候也可以配置按照某种条件删除（condition属性）或者或者配置清除所有缓存（allEntries属性），示例代码如下：

```java
import org.springframework.cache.annotation.CacheEvict;

@Override
@CacheEvict(key = "#id")
public int delete(Integer id) {
    return sysLogMapper.delete(id);
}
```

### 缓存注解说明

1. `@CacheConfig`：主要用于配置该类中会用到的一些共用的缓存配置。

   在这里`@CacheConfig(cacheNames = "sysLog")`：配置了该数据访问对象中返回的内容将存储于名为sysLog的缓存对象中，我们也可以不使用该注解，直接通过`@Cacheable`自己配置缓存集的名字来定义；

2. `@Cacheable`：配置了`querySysLogById`函数的返回值将被加入缓存。同时在查询时，会先从缓存中获取，若不存在才再发起对数据库的访问。该注解主要有下面几个参数：

   - `value`、`cacheNames`：两个等同的参数（cacheNames为Spring 4新增，作为value的别名），用于指定缓存存储的集合名。由于Spring 4中新增了`@CacheConfig`，因此在Spring 3中原本必须有的value属性，也成为非必需项了；
   - `key`：缓存对象存储在Map集合中的key值，非必需，缺省按照函数的所有参数组合作为key值，若自己配置需使用SpEL表达式，比如：`@Cacheable(key = "#p0")`：使用函数第一个参数作为缓存的key值，更多关于SpEL表达式的详细内容可[参考地址](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#cache)；
   - `condition`：缓存对象的条件，非必需，也需使用SpEL表达式，只有满足表达式条件的内容才会被缓存，比如：`@Cacheable(key = "#p0", condition = "#p0.length() < 3")`，表示只有当第一个参数的长度小于3的时候才会被缓存；
   - `unless`：另外一个缓存条件参数，非必需，需使用SpEL表达式。它不同于condition参数的地方在于它的判断时机，该条件是在函数被调用之后才做判断的，所以它可以通过对result进行判断；
   - `keyGenerator`：用于指定key生成器，非必需。若需要指定一个自定义的key生成器，我们需要去实现`org.springframework.cache.interceptor.KeyGenerator`接口，并使用该参数来指定；
   - `cacheManager`：用于指定使用哪个缓存管理器，非必需。只有当有多个时才需要使用；
   - `cacheResolver`：用于指定使用那个缓存解析器，非必需。需通过org.springframework.cache.interceptor.CacheResolver接口来实现自己的缓存解析器，并用该参数指定；

3. `@CachePut`：配置于函数上，能够根据参数定义条件来进行缓存，其缓存的是方法的返回值，它与`@Cacheable`不同的是，它每次都会真实调用函数，所以主要用于数据新增和修改操作上。它的参数与`@Cacheable`类似，具体功能可参考上面对`@Cacheable`参数的解析；

4. `@CacheEvict`：配置于函数上，通常用在删除方法上，用来从缓存中移除相应数据。除了同`@Cacheable`一样的参数之外，它还有下面两个参数：

   - `allEntries`：非必需，默认为false。当为true时，会移除所有数据；
   - `beforeInvocation`：非必需，默认为false，会在调用方法之后移除数据。当为true时，会在调用方法之前移除数据。

### 缓存实现

要使用上Spring Boot的缓存功能，还需要提供一个缓存的具体实现。Spring Boot根据下面的顺序去侦测缓存实现：

- Generic
- JCache (JSR-107)
- EhCache 2.x
- Hazelcast
- Infinispan
- Redis
- Guava
- Simple

除了按顺序侦测外，我们也可以通过配置属性`spring.cache.type`来强制指定。

接下来主要介绍基于 Redis 和 Ehcache 的缓存实现。

### Redis 的缓存实现

Redis的[下载地址](https://github.com/MicrosoftArchive/redis/releases)，Redis 支持 32 位和 64 位。

启动redis的服务， 进入redis的目录执行命令`redis-server.exe redis.windows.conf`

引入redis的pom依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

在`application.yml`文件中配置Redis

```yaml
spring:
  redis:
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: localhost
    # Redis服务器连接端口
    port: 6379
    # 连接超时时间（毫秒） 
    # 如果超时时间设置为0时会报错：Command timed out after no timeout 
    timeout: 50
  cache:
    type: redis
```

更多[Spring Boot Redis配置](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html#%20REDIS)

运行测试，控制台输出：

```java
2019-08-14 14:31:29.932 DEBUG 32920 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : ==>  Preparing: SELECT * FROM sys_log WHERE id = ? 
2019-08-14 14:31:29.965 DEBUG 32920 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : ==> Parameters: 2(Integer)
2019-08-14 14:31:30.011 DEBUG 32920 --- [           main] c.t.s.m.SysLogMapper.querySysLogById     : <==      Total: 1
SysLog{id=2, username='tengxt', operation='执行方法二', time='2001', method='com.tengxt.springbootaoplog.controller.SysLogController.methodTwo()', ip='127.0.0.1', createTime=Tue Aug 06 08:00:00 CST 2019, params=''}
SysLog{id=2, username='tengxt', operation='执行方法二', time='2001', method='com.tengxt.springbootaoplog.controller.SysLogController.methodTwo()', ip='127.0.0.1', createTime=Tue Aug 06 08:00:00 CST 2019, params=''}
```

可发现第一次查询查的是数据库，第二次查询是Redis缓存

### Ehcache 的缓存实现

引入redis的pom依赖

```xml
<!-- ehcache -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

在`application.yml`中指定ehcache配置的路径

```yaml
spring:
  cache:
    # 缓存类型
    type: ehcache
    ehcache:
      config: 'classpath:ehcache.xml'
```

在`src/main/resources`目录下新建ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="3600"
            timeToLiveSeconds="0"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120" />
<!--注意： name="sysLog" 与缓存类中的 @CacheConfig(cacheNames = "sysLog") 的名称应保持一致 -->
    <cache
            name="sysLog"
            maxEntriesLocalHeap="2000"
            eternal="false"
            timeToIdleSeconds="3600"
            timeToLiveSeconds="0"
            overflowToDisk="false"
            statistics="true"/>
</ehcache>
```

关于Ehcahe的一些说明：

- name：缓存名称。
- maxElementsInMemory：缓存最大数目
- maxElementsOnDisk：硬盘最大缓存个数。
- eternal：对象是否永久有效，一但设置了，timeout将不起作用。
- overflowToDisk：是否保存到磁盘。
- timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当`eternal=false`对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
- timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当`eternal=false`对象不是永久有效时使用，默认是0，也就是对象存活时间无穷大。
- diskPersistent：是否缓存虚拟机重启期数据，默认值为false。
- diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
- diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
- memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
- clearOnFlush：内存数量最大时是否清除。
- memoryStoreEvictionPolicy：Ehcache的三种清空策略：**FIFO**，first in first out，这个是大家最熟的，先进先出。**LFU**， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。**LRU**，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。

### 总结

在Spring Boot中，使用Redis缓存，既可以使用RedisTemplate自己来实现，也可以使用Spring Cache提供的统一接口，实现既可以是Redis，也可以是Ehcache或者其他支持这种规范的缓存框架。从这个角度来说，Spring Cache和Redis、Ehcache的关系就像JDBC与各种数据库驱动的关系。

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-redis-cache)



















