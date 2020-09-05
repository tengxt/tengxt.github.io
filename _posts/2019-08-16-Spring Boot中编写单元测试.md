---
layout:     post
title:      "Spring Boot中编写单元测试"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-16 22:57:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - JUnit4
---

编写单元测试可以帮助开发人员编写高质量的代码，提升代码质量，减少Bug，便于重构。Spring Boot提供了一些实用程序和注解，用来帮助我们测试应用程序，在Spring Boot中开启单元测试只需引入`spring-boot-starter-test`即可，其包含了一些主流的测试库。

引入`spring-boot-starter-test`：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

在项目根路径下执行命令`mvn dependency:tree`可看到其包含了以下关于测试的依赖

```powershell
[INFO] \- org.springframework.boot:spring-boot-starter-test:jar:2.1.7.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test:jar:2.1.7.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test-autoconfigure:jar:2.1.7.RELEASE:test
[INFO]    +- com.jayway.jsonpath:json-path:jar:2.4.0:test
[INFO]    |  +- net.minidev:json-smart:jar:2.3:test
[INFO]    |  |  \- net.minidev:accessors-smart:jar:1.2:test
[INFO]    |  |     \- org.ow2.asm:asm:jar:5.0.4:test
[INFO]    |  \- org.slf4j:slf4j-api:jar:1.7.26:compile
[INFO]    +- junit:junit:jar:4.12:test
[INFO]    +- org.assertj:assertj-core:jar:3.11.1:test
[INFO]    +- org.mockito:mockito-core:jar:2.23.4:test
[INFO]    |  +- net.bytebuddy:byte-buddy:jar:1.9.16:test
[INFO]    |  +- net.bytebuddy:byte-buddy-agent:jar:1.9.16:test
[INFO]    |  \- org.objenesis:objenesis:jar:2.6:test
[INFO]    +- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO]    +- org.hamcrest:hamcrest-library:jar:1.3:test
[INFO]    +- org.skyscreamer:jsonassert:jar:1.5.0:test
[INFO]    |  \- com.vaadin.external.google:android-json:jar:0.0.20131108.vaadin1:test
[INFO]    +- org.springframework:spring-core:jar:5.1.9.RELEASE:compile
[INFO]    |  \- org.springframework:spring-jcl:jar:5.1.9.RELEASE:compile
[INFO]    +- org.springframework:spring-test:jar:5.1.9.RELEASE:test
[INFO]    \- org.xmlunit:xmlunit-core:jar:2.6.3:test
```

- JUnit，标准的单元测试Java应用程序；
- Spring Test & Spring Boot Test，对Spring Boot应用程序的单元测试提供支持；
- Mockito, Java mocking框架，用于模拟任何Spring管理的Bean，比如在单元测试中模拟一个第三方系统Service接口返回的数据，而不会去真正调用第三方系统；
- AssertJ，一个流畅的assertion库，同时也提供了更多的期望值与测试返回值的比较方式；
- Hamcrest，库的匹配对象（也称为约束或谓词）；
- JsonPath，提供类似XPath那样的符号来获取JSON数据片段；
- JSONassert，对JSON对象或者JSON字符串断言的库。

一个标准的Spring Boot测试单元应有如下的代码结构：

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTestingApplicationTests {
    @Test
    public void contextLoads() {}
}
```

### 知识准备

#### JUnit4注解

JUnit4中包含了几个比较重要的注解：`@BeforeClass`、`@AfterClass`、`@Before`、`@After`和`@Test`。其中， `@BeforeClass`和`@AfterClass`在每个类加载的开始和结束时运行，必须为静态方法；而`@Before`和`@After`则在每个测试方法开始之前和结束之后运行。

运行结果如下：

```java
...
before class test
before test
test 1+1=2
after test
before test
test 2+2=4
after test
after class test
...
```

从上面的输出可以看出各个注解的运行时机。

#### Assert

Assert类提供的assert口方法，下面列出了一些常用的assert方法：

- `assertEquals("message",A,B)`，判断A对象和B对象是否相等，这个判断在比较两个对象时调用了`equals()`方法。
- `assertSame("message",A,B)`，判断A对象与B对象是否相同，使用的是`==`操作符。
- `assertTrue("message",A)`，判断A条件是否为真。
- `assertFalse("message",A)`，判断A条件是否不为真。
- `assertNotNull("message",A)`，判断A对象是否不为`null`。
- `assertArrayEquals("message",A,B)`，判断A数组与B数组是否相等。

```java
@Test
public void test1() {
    System.out.println("test 1 + 1 = 2");
    Assert.assertEquals(2, 1 + 1);
}
```

#### MockMvc

对Controller的测试需要用到MockMvc技术。MockMvc，从字面上来看指的是模拟的MVC，即其可以模拟一个MVC环境，向Controller发送请求然后得到响应。

在单元测试中，使用MockMvc前需要进行初始化。

```java
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTestingApplicationTests {

    @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;

    @Before
    public void setupMockMvc(){
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }
}
```

**MockMvc模拟MVC请求**

模拟一个get请求：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/hello?name={name}","tengxt"));
```

模拟一个post请求：

```java
mockMvc.perform(MockMvcRequestBuilders.post("/user/{id}", 1));
```

模拟文件上传：

```java
mockMvc.perform(MockMvcRequestBuilders.fileUpload("/fileupload").file("file", "文件内容".getBytes("utf-8")));
```

模拟请求参数：

```java
// 模拟发送一个message参数，值为hello
mockMvc.perform(MockMvcRequestBuilders.get("/hello").param("message", "hello"));
// 模拟提交一个checkbox值，name为hobby，值为sleep和eat
mockMvc.perform(MockMvcRequestBuilders.get("/saveHobby").param("hobby", "sleep", "eat"));
```

也可以直接使用`MultiValueMap`构建参数：

```java
MultiValueMap<String, String> params = new LinkedMultiValueMap<String, String>();
params.add("name", "mrbird");
params.add("hobby", "sleep");
params.add("hobby", "eat");
mockMvc.perform(MockMvcRequestBuilders.get("/hobby/save").params(params));
```

模拟发送JSON参数：

```java
String jsonStr = "{\"username\":\"Dopa\",\"passwd\":\"ac3af72d9f95161a502fd326865c2f15\",\"status\":\"1\"}";
mockMvc.perform(MockMvcRequestBuilders.post("/user/save").content(jsonStr.getBytes()));
```

实际测试中，要手动编写这么长的JSON格式字符串很繁琐也很容易出错，可以借助Spring Boot自带的Jackson技术来序列化一个Java对象（可参考[Spring Boot中的JSON技术](https://tengxt.gitee.io/2019/08/15/Spring-Boot%E4%B8%AD%E7%9A%84JSON%E6%8A%80%E6%9C%AF/)），如下所示：

```java
User user = new User();
user.setUsername("Dopa");
user.setPasswd("ac3af72d9f95161a502fd326865c2f15");
user.setStatus("1");

String userJson = mapper.writeValueAsString(user);
mockMvc.perform(MockMvcRequestBuilders.post("/user/save").content(userJson.getBytes()));
```

其中，mapper为`com.fasterxml.jackson.databind.ObjectMapper`对象。

模拟Session和Cookie：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/index").sessionAttr(name, value));
mockMvc.perform(MockMvcRequestBuilders.get("/index").cookie(new Cookie(name, value)));
```

设置请求的Content-Type：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/index").contentType(MediaType.APPLICATION_JSON_UTF8));
```

设置返回格式为JSON：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1).accept(MediaType.APPLICATION_JSON));
```

模拟HTTP请求头：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1).header(name, values));
```

**MockMvc处理返回结果**

期望成功调用，即HTTP Status为200：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1))
    .andExpect(MockMvcResultMatchers.status().isOk());
```

期望返回内容是`application/json`：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1))
    .andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON));
```

检查返回JSON数据中某个值的内容：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1))
    .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("mrbird"));
```

这里使用到了`jsonPath`，`$`代表了JSON的根节点。更多关于`jsonPath`的介绍可[参考文档](https://github.com/json-path/JsonPath) 

判断Controller方法是否返回某视图：

```java
mockMvc.perform(MockMvcRequestBuilders.post("/index"))
    .andExpect(MockMvcResultMatchers.view().name("index.html"));
```

比较Model：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", 1))
    .andExpect(MockMvcResultMatchers.model().size(1))
    .andExpect(MockMvcResultMatchers.model().attributeExists("password"))
    .andExpect(MockMvcResultMatchers.model().attribute("username", "mrbird"));
```

比较forward或者redirect：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andExpect(MockMvcResultMatchers.forwardedUrl("index.html"));
// 或者
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andExpect(MockMvcResultMatchers.redirectedUrl("index.html"));
```

比较返回内容，使用`content()`：

```java
// 返回内容为hello
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andExpect(MockMvcResultMatchers.content().string("hello"));

// 返回内容是XML，并且与xmlCotent一样
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andExpect(MockMvcResultMatchers.content().xml(xmlContent));

// 返回内容是JSON ，并且与jsonContent一样
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andExpect(MockMvcResultMatchers.content().json(jsonContent));
```

输出响应结果：

```java
mockMvc.perform(MockMvcRequestBuilders.get("/index"))
    .andDo(MockMvcResultHandlers.print());
```

### 测试Service

```java
@Service("sysLogService")
public class SysLogServiceImpl implements SysLogService {
    
    @Autowired
    private SysLogMapper sysLogMapper;

    @Override
    public SysLog findByName(String username) {
        SysLog sysLog = null;
        if(null != username && !"".equals(username)){
            List<SysLog> sysLogs = sysLogMapper.findByName(username);
            if(null != sysLogs && sysLogs.size() > 0){
                sysLog =  sysLogs.get(0);
            }
        }
        return sysLog;
    }
}
```

编写一个该Service的单元测试，测试`findByName`方法是否有效

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SysLogServiceTest {

    @Autowired
    private SysLogService sysLogService;

    @Test
    public void Test(){
        SysLog sysLog = sysLogService.findByName("tengxt");
        Assert.assertEquals("用户名为：tengxt", "tengxt", sysLog.getUsername());
    }
}
```

运行后，JUnit没有报错说明测试通过，即`UserService`的`findByName`方法可行。

此外，和在Controller中引用Service相比，在测试单元中对Service测试完毕后，数据能自动回滚，只需要在测试方法上加上`@Transactional`注解。

```java
@Test
@Transactional
public void save(){
    SysLog sysLog = new SysLog();
    sysLog.setUsername("tengxt111");
    sysLog.setOperation("测试");
    sysLog.setTime("111");
    sysLog.setMethod("com.tengxt.springboottesting.service.SysLogServiceTest.save");
    sysLog.setIp("127.0.0.1");
    sysLog.setCreateTime(new Date());
    sysLog.setParams("");
    Integer res = sysLogService.save(sysLog);
    System.out.println("受影响行数:" + res);
}
```

运行，测试通过，查看数据库发现数据并没有被插入，这样很好的避免了不必要的数据污染。

### 测试Controller

```java
@RestController
public class SysLogController {
    @Autowired
    private SysLogService sysLogService;

    @GetMapping("log/{userName}")
    public SysLog getUserByName(@PathVariable(value = "userName") String userName) {
        return this.sysLogService.findByName(userName);
    }

    @PostMapping("user/save")
    public int save(@RequestBody SysLog log) {
        return this.sysLogService.save(log);
    }
}
```

针对于该Controller`getUserByName(@PathVariable(value = "userName") String userName)`方法的测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SysLogControllerTest {
    private MockMvc mockMvc;

    @Autowired
    private WebApplicationContext wac;

    @Before
    public void setupMockMvc(){
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void findByName() throws Exception{
        mockMvc.perform(
                MockMvcRequestBuilders.get("/log/{userName}", "tengxt")
                        .contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("tengxt"))
                .andDo(MockMvcResultHandlers.print());
    }
}
```

运行后控制台输出过程

```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /log/tengxt
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8"]
             Body = null
    Session Attrs = {}

Handler:
             Type = com.tengxt.springboottesting.controller.SysLogController
           Method = public com.tengxt.springboottesting.entity.SysLog com.tengxt.springboottesting.controller.SysLogController.getUserByName(java.lang.String)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"id":2,"username":"tengxt","operation":"执行方法二","time":"2001","method":"com.tengxt.springbootaoplog.controller.SysLogController.methodTwo()","ip":"127.0.0.1","createTime":"2019-08-06T00:00:00.000+0000","params":""}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-testing)


















