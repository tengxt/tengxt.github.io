---
layout:     post
title:      "使用Spring Boot发送邮件"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-21 23:33:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
    - Email
---


> 环境： Spring Boot 2.1.7.RELEASE 、Java 8

在项目的维护过程中，我们通常会在应用中加入短信或者邮件预警功能，比如当应用出现异常宕机时应该及时地将预警信息发送给运维或者开发人员，本文将介绍如何在Spring Boot中发送邮件。在Spring Boot中发送邮件使用的是Spring提供的`org.springframework.mail.javamail.JavaMailSender`，其提供了许多简单易用的方法，可发送简单的邮件、HTML格式的邮件、带附件的邮件，并且可以创建邮件模板。

### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### 邮件配置

在`application.properties`中进行简单的配置（以163邮件为例）

```properties
## 发送邮件配置
# 163邮箱的smtp
spring.mail.host=smtp.163.com
# 端口
spring.mail.port=25
# 邮箱的用户名
spring.mail.username=xxxx@163.com
# 邮箱的授权码（不是登录密码）
spring.mail.password=xxxx
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

163邮箱的收取邮件支持POP/IMAP两种协议，发送邮件采用SMTP协议，收件和发件均使用SSL协议来进行加密传输，采用SSL协议需要单独对帐户进行设置。采用SSL协议和非SSL协议时端口号有所区别，参照下表的一些常见配置组合

| 类型       | 服务器名称 | 服务器地址   | SSL协议端口号 | 非SSL协议端口号 |
| ---------- | ---------- | ------------ | ------------- | --------------- |
| 收件服务器 | POP        | pop.163.com  | 995           | 110             |
| 收件服务器 | IMAP       | imap.163.com | 993           | 143             |
| 发件服务器 | SMTP       | smtp.163.com | 465/994       | 25              |

> 163邮箱的地址和端口： <http://help.163.com/10/0731/11/6CTUBPT300753VB8.html>

### 发送简单的邮件

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.MailException;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 发送简单的邮件
 */
@RestController
@RequestMapping("/email")
public class EmailController {
    @Autowired
    private JavaMailSender javaMailSender;

    @Value("${spring.mail.username}")
    private String from;

    @RequestMapping("/sendSimpleEmail")
    public String sendSimpleEmail(){
        try {
            SimpleMailMessage mailMessage = new SimpleMailMessage();
            mailMessage.setFrom(from);
            // 接收地址
            mailMessage.setTo("xxxx@qq.com");
            // 标题
            mailMessage.setSubject("一封简单的邮件");
            // 内容
            mailMessage.setText("使用Spring Boot发送简单邮件。");
            javaMailSender.send(mailMessage);
            return "发送成功";
        } catch (MailException e) {
            e.printStackTrace();
        }
        return "发送失败";
    }
}
```

启动项目访问<http://localhost:8080/email/sendSimpleEmail>，提示发送成功。

### 发送带有HTML格式的邮件

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

@RestController
@RequestMapping("/email")
public class HtmlEmailController {
    @Autowired
    private JavaMailSender javaMailSender;

    @Value("${spring.mail.username}")
    private String from;

    @RequestMapping("/sendHtmlEmail")
    public String sendHtmlEmail(){
        MimeMessage message = null;
        try {
            message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            // 接收地址
            helper.setTo("xxxx@qq.com");
            // 标题
            helper.setSubject("一封带有<b>HTML</b>格式的邮件");
            // 带有HTML格式的内容
            StringBuffer stringBuffer = new StringBuffer("<p>使用<b>SpringBoot</b>发送<strong>HTML</strong>格式的邮件</p>");
            helper.setText(stringBuffer.toString(), true);
            javaMailSender.send(message);
            return "发送成功";
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return "发送失败";
    }
}
```

启动项目访问<http://localhost:8080/email/sendHtmlEmail>，提示发送成功。

### 发送带有附件的邮件

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@RestController
@RequestMapping("/email")
public class AttachEmailController {
    @Autowired
    private JavaMailSender javaMailSender;

    @Value("${spring.mail.username}")
    private String from;

    @RequestMapping("/sendAttachEmail")
    public String sendAttachMail(){
        MimeMessage message = null;
        try {
            message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            // 接收地址
            helper.setTo("xxxx@qq.com");
            // 标题
            helper.setSubject("一封带附件的邮件");
            // 内容
            helper.setText("详情参考附件内容");
            // 传入附件
            FileSystemResource fileSystemResource =
                    new FileSystemResource(new File("src\\main\\resources\\static\\附件.docx"));
            helper.addAttachment("附件.docx", fileSystemResource);
            javaMailSender.send(message);
            return "发送成功";
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return "发送失败";
    }
}
```

启动项目访问<http://localhost:8080/email/sendAttachEmail>，提示发送成功。

### 发送带静态资源的邮件

发送带静态资源的邮件其实就是在发送HTML邮件的基础上嵌入静态资源（比如图片），嵌入静态资源的过程和传入附件类似，唯一的区别在于需要标识资源的cid。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@RestController
@RequestMapping("/email")
public class ImgEmailController {
    @Autowired
    private JavaMailSender javaMailSender;

    @Value("${spring.mail.username}")
    private String from;

    @RequestMapping("/sendImgMail")
    public String sendImgMail(){
        MimeMessage message = null;
        try {
            message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            // 接收地址
            helper.setTo("1300230407@qq.com");
            // 标题
            helper.setSubject("一封带有静态资源的邮件");
            // 内容
            helper.setText("<html><body>这是一张图片：<img src='cid:img' width='50%' height='50%'/></body></html>", true);
            // 传入附件
            FileSystemResource fileSystemResource =
                    new FileSystemResource(new File("src\\main\\resources\\static\\img\\6f8a2832ly1fw4n2oxwmgj21hc0u07he.jpg"));
            // 把图片绑定到cid为img上
            helper.addInline("img", fileSystemResource);
            javaMailSender.send(message);
            return "发送成功";
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return "发送失败";
    }
}
```

`helper.addInline("img", file);`中的img和图片标签里cid后的名称相对应。启动项目访问<http://localhost/email/sendImgMail>，提示发送成功。

### 使用模板发送邮件

在发送验证码等情况下可以创建一个邮件的模板，唯一的变量为验证码。这个例子中使用的模板解析引擎为Thymeleaf，所以首先引入Thymeleaf依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

在template目录下创建一个`emailTemplate.html`模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>发送模板</title>
</head>
<body>
    <p>您好，您的验证码为：<span th:text="${code}"></span>，请在两分钟内使用完成操作。</p>
</body>
</html>
```

发送模板邮件，本质上还是发送HTML邮件，只不过多了绑定变量的过程。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

@RestController
@RequestMapping("/email")
public class TempEmailController {
    @Autowired
    private JavaMailSender javaMailSender;

    @Autowired
    private TemplateEngine templateEngine;

    @Value("${spring.mail.username}")
    private String from;

    @RequestMapping("/sendTempMail")
    public String sendTempMail(String code){
        MimeMessage message = null;
        try {
            message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            // 接收地址
            helper.setTo("1300230407@qq.com");
            // 标题
            helper.setSubject("一封带有静态资源的邮件");
            // 出来邮件模板
            Context context = new Context();
            context.setVariable("code", code);
            String template = templateEngine.process("emailTemplate", context);
            helper.setText(template, true);
            javaMailSender.send(message);
            return "发送成功";
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return "发送失败";
    }
}
```

其中`code`对应模板里的`${code}`变量。启动项目，访问<http://localhost:8080/email/sendTempMail?code=abc2>，页面提示发送成功。

> [source code](https://github.com/tengxt/springboot-learn/tree/master/spring-boot-email)













