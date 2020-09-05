---
layout:     post
title:      "Spring Boot中的JSON技术"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-15 21:14:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - JSON
---

平日里在项目中处理JSON一般用的都是阿里巴巴的Fastjson，后来发现使用Spring Boot内置的Jackson来完成JSON的序列化和反序列化操作也挺方便。Jackson不但可以完成简单的序列化和反序列化操作，也能实现复杂的个性化的序列化和反序列化操作。

## 自定义ObjectMapper

我们都知道，在Spring中使用`@ResponseBody`注解可以将方法返回的对象序列化成JSON

```java
import com.tenegxt.springbootjackson.entity.User;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Date;

@RestController  //  @Controller + @ResponseBody
public class UserController {

    @RequestMapping("/getUser")
    public User getUser(){
        User user = new User();
        user.setUserName("tengxt");
        user.setAge(20);
        user.setBirthday(new Date());
        return user;
    }
}
```

进行访问`http://localhost:8080/getUser`，得到的数据如下：

```json
{"id":null,"userName":"tengxt","password":null,"age":20,"birthday":"2019-08-15T03:18:06.842+0000"}
```

可看到时间的格式并不好理解，如果想要改变这个默认行为，我们可以自定义一个ObjectMapper来替代

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.text.SimpleDateFormat;

@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper getObjectMapper(){
        ObjectMapper mapper = new ObjectMapper();
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return mapper;
    }
}
```

进行访问`http://localhost:8080/getUser` ，得到的数据如下：

```json
{"id":null,"userName":"tengxt","password":null,"age":20,"birthday":"2019-08-15 11:35:01"}
```

## 序列化

Jackson通过使用mapper的`writeValueAsString`方法将Java对象序列化为JSON格式字符串

```java
import com.tenegxt.springbootjackson.entity.User;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.tenegxt.springbootjackson.entity.User;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Date;

@RestController  //  @Controller + @ResponseBody
public class UserController {
    @Autowired
    private ObjectMapper mapper;

    @GetMapping("/getSerUser")
    public String getSerializationUser(){
        String res= null;
        try {
            User user = new User();
            user.setUserName("tengxt");
            user.setAge(20);
            user.setBirthday(new Date());
            res = mapper.writeValueAsString(user);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return res;
    }
}
```

## 反序列化

Jackson也提供了反序列化方法

### 树遍历

当采用树遍历的方式时，JSON被读入到`JsonNode`对象中，可以像操作XML DOM那样读取JSON。

```java
@GetMapping("/readJsonString")
public String readJsonString(){
    try {
        String json ="{\"userName\":\"tengxt\",\"age\":20}";
        JsonNode node = this.mapper.readTree(json);
        String userName = node.get("userName").asText();
        int age = node.get("age").asInt();
        return userName + " " + age;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

`readTree`方法可以接受一个字符串或者字节数组、文件、InputStream等， 返回`JsonNode`作为根节点。

解析多级JSON例子

```java
String json="{\"userName\":\"tengxt\",\"hobby\":{\"first\":\"sleep\",\"second\":\"eat\"}}";
JsonNode jsonNode = this.mapper.readTree(json);
JsonNode hobby = jsonNode.get("hobby");
String firstStr = hobby.get("first").asText();
String secondStr = hobby.get("second").asText();
```

### 绑定对象

可以将Java对象和JSON数据进行绑定。

```java
public String readJsonAsObject(){
    try {
        String json ="{\"userName\":\"tengxt\",\"age\":20}";
        User user = mapper.readValue(json, User.class);
        String userName = user.getUserName();
        Integer age = user.getAge();
        return userName + " " + age;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

## Jackson 注解

### @JsonProperty

`@JsonProperty`，作用在属性上，用来为JSON Key指定一个别名。

```java
@JsonProperty("bth")
private Date birthday;
```

再次访问`getuser`页面输出：

```json
{"id":null,"userName":"tengxt","password":null,"age":20,"btn":"2019-08-15 15:16:13"}
```

key birthday已经被替换为了bth。

### @Jsonlgnore

`@Jsonlgnore`，作用在属性上，用来忽略此属性。

```java
@JsonIgnore
private String password;
```

再次访问`getuser`页面输出：

```json
{"id":null,"userName":"tengxt","age":20,"btn":"2019-08-15 15:18:59"}
```

password属性已被忽略。

### @JsonIgnoreProperties

`@JsonIgnoreProperties`，忽略一组属性，作用于类上，比如`@JsonIgnoreProperties({"id","password"})`。

```java
@JsonIgnoreProperties({ "id", "password" })
public class User implements Serializable {
    ...
}
```

再次访问`getuser`页面输出：

```json
{"userName":"tengxt","age":20,"btn":"2019-08-15 15:21:22"}
```

### @JsonFormat

`@JsonFormat`，用于日期格式化，如：

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private Date birthday;
```

### @JsonNaming

`@JsonNaming`，用于指定一个命名策略，作用于类或者属性上。Jackson自带了多种命名策略，你可以实现自己的命名策略，比如输出的key 由Java命名方式转为下面线命名方法 —— userName转化为user-name。

```java
@JsonNaming(PropertyNamingStrategy.LowerCaseWithUnderscoresStrategy.class)
public class User implements Serializable {
    ...
}
```

再次访问`getuser`页面输出：

```json
{"user_name":"tengxt","age":20,"btn":"2019-08-15 07:35:12"}
```

### @JsonSerialize

`@JsonSerialize`，指定一个实现类来自定义序列化。类必须实现`JsonSerializer`接口，代码如下：

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.tenegxt.springbootjackson.entity.User;

import java.io.IOException;

public class UserSerializer extends JsonSerializer<User> {

    @Override
    public void serialize(User user, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField("user-name", user.getUserName());
        jsonGenerator.writeEndObject();
    }
}
```

上面的代码中我们仅仅序列化userName属性，且输出的key是`user-name`。 使用注解`@JsonSerialize`来指定User对象的序列化方式：

```java
@JsonSerialize(using = UserSerializer.class)
public class User implements Serializable {
    ...
}
```

再次访问`getuser`页面输出：

```json
{"user-name":"tengxt"}
```

### @JsonView

`@JsonView`，作用在类或者属性上，用来定义一个序列化组。 比如对于User对象，某些情况下只返回userName属性就行，而某些情况下需要返回全部属性。 

```java
public class User implements Serializable {
    private static final long serialVersionUID = -5617919495946624537L;

    public interface UserNameView {};
    public interface AllUserFieldView extends UserNameView {};

    @JsonView(AllUserFieldView.class)
    private Integer id;
    
    @JsonView(UserNameView.class)
    private String userName;
    
    @JsonView(AllUserFieldView.class)
    private String password;
    
    @JsonView(AllUserFieldView.class)
    private Integer age;
    
    @JsonView(AllUserFieldView.class)
    private Date birthday;
    // ...
}
```

User定义了两个接口类，一个为`userNameView`，另外一个为`AllUserFieldView`继承了`userNameView`接口。这两个接口代表了两个序列化组的名称。属性userName使用了`@JsonView(UserNameView.class)`，而剩下属性使用了`@JsonView(AllUserFieldView.class)`。

Spring中Controller方法允许使用`@JsonView`指定一个组名，被序列化的对象只有在这个组的属性才会被序列化，代码如下：

```java
@GetMapping("/getUser")
@JsonView(User.UserNameView.class)
public User getUser(){
    User user = new User();
    user.setUserName("tengxt");
    user.setAge(20);
    user.setBirthday(new Date());
    return user;
}
```

访问`getuser`页面输出：

```
{"userName":"tengxt"}
```

如果将`@JsonView(User.UserNameView.class)`替换为`@JsonView(User.AllUserFieldView.class)`，输出：

```json
{"id":null,"userName":"tengxt","password":null,"age":20,"birthday":"2019-08-15 16:10:01"}
```

因为接口`AllUserFieldView`继承了接口`UserNameView`所以userName也会被输出。

## 集合的反序列化

在Controller方法中，可以使用`＠RequestBody`将提交的JSON自动映射到方法参数上，比如：

```java
@PostMapping("/updateUser")
public int updateUser(@RequestBody List<User> users){
    System.out.println(users);
    return users.size();
}
```

上面方法可以接收`JSON`请求，并自动映射到User对象上：

```
[{"userName":"tengxt1","age":20},{"userName":"tengxt2","age":22}]
```

Spring Boot 能自动识别出List对象包含的是User类，因为在方法中定义的泛型的类型会被保留在字节码中，所以Spring Boot能识别List包含的泛型类型从而能正确反序列化。

有些情况下，集合对象并没有包含泛型定义，如下代码所示，反序列化并不能得到期望的结果。

```java
@Autowired
ObjectMapper mapper;

@RequestMapping("customize")
@ResponseBody
public String customize() throws JsonParseException, JsonMappingException, IOException {
    String jsonStr = "[{\"userName\":\"mrbird\",\"age\":26},{\"userName\":\"scott\",\"age\":27}]";
    List<User> list = mapper.readValue(jsonStr, List.class);
    String msg = "";
    for (User user : list) {
        msg += user.getUserName();
    }
    return msg;
}
```

访问`http://localhost:8080/customize`，控制台抛出异常：

```powershell
java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to com.tenegxt.springbootjackson.entity.User
```

这是因为在运行时刻，泛型己经被擦除了（不同于方法参数定义的泛型，不会被擦除）。为了提供泛型信息，Jackson提供了JavaType ，用来指明集合类型，将上述方法改为：

```java
@GetMapping("customize")
    public String customize() throws JsonParseException, JsonMappingException, IOException {
        String jsonStr = "[{\"userName\":\"tengxt1\",\"age\":21},{\"userName\":\"tengxt2\",\"age\":22}]";
        //List<User> list = mapper.readValue(jsonStr, List.class);
        // 抛出异常： java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to com.tenegxt.springbootjackson.entity.User
        JavaType type = mapper.getTypeFactory().constructParametricType(List.class, User.class);
        List<User> list = mapper.readValue(jsonStr, type);
        String msg = "";
        for (User user : list) {
            msg += user.getUserName();
        }
        return msg;
    }
```

访问`customize`，页面输出：`tengxt1tengxt2`。
















