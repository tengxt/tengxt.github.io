---
layout:     post
title:      "Spring Boot应用的后台运行配置"
subtitle:   "Spring Boot 2.1.6"
date:       2019-08-28 22:02:00
author:     "Xt"
header-style: text
tags:
    - Spring Boot 2.1.6
---

> 环境    Java 8   Spring Boot  2.1.7.RELEASE

Spring Boot应用的几种运行方式

- 运行Spring Boot的应用主类
- 使用Maven的Spring Boot插件`mvn spring-boot:run`来运行
- 打成jar包后，使用`java -jar`运行

在开发的时候通常会使用前两种，而在部署的时候往往会使用第三种。但是，我们在使用`java -jar`来运行的时候，但这样做无法将shell命令行释放，关闭terminal终端后项目也随之关闭了。

### nohup和Shell

> nohup 命令
>
> 用途：不挂断地运行命令。
>
> 语法：nohup Command [ Arg … ][ & ]
>
> 描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 `&`到命令的尾部。

所以，我们只需要使用`nohup java -jar yourapp.jar &`命令，就能让`yourapp.jar`在后台运行了。但是，为了方便管理，我们还可以通过Shell来编写一些用于启动应用的脚本。

**关闭应用的脚本**：`stop.sh`

```powershell
#!/bin/bash
PID=$(ps -ef | grep yourapp.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    echo Application is already stopped
else
    echo kill $PID
    kill $PID
fi
```

**启动应用的脚本**：`start.sh`

```powershell
#!/bin/bash
nohup java -jar yourapp.jar --server.port=8888 &
```

**整合了关闭和启动的脚本**：`run.sh`

由于会先执行关闭应用，然后再启动应用，这样不会引起端口冲突等问题，适合在持续集成系统中进行反复调用。

```powershell
#!/bin/bash
echo stop application
source stop.sh
echo start application
source start.sh
```

**在编写shell脚本的过程中遇到了两个问题**：

- 执行`.sh`文件提示权限不足

  解决办法：执行命令`chmod u+x XX.sh`赋予当前用于可执行的权限即可。

- 提示/bin/bash^M: bad interpreter: 没有那个文件或目录

  问题出现的原因是shell脚本是在windows中编写的然后上传到Linux中的，出现了兼容性问题。解决办法：执行`vim XX.sh`打开shell文件，然后切换到命令模式，执行`:set fileformat=unix`后保存退出即可。

- 使用了`nohup`命令后，会在jar文件目录下生成一个nohup.out文件

  ```powershell
   # 查看日志
   tail -f nohup.out 
  ```

### 系统服务

在Spring Boot的Maven插件中，还提供了构建完整可执行程序的功能，什么意思呢？就是说，我们可以不用`java -jar`，而是直接运行jar来执行程序。这样我们就可以方便的将其创建成系统服务在后台运行了。主要步骤如下：

- 在`pom.xml`中添加Spring Boot的插件，并注意设置`executable`配置

  ```xml
  <build> 
    <plugins> 
      <plugin> 
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-maven-plugin</artifactId>  
        <configuration> 
          <executable>true</executable> 
        </configuration> 
      </plugin> 
    </plugins> 
  </build>
  ```

- 在完成上述配置后，使用`mvn install`进行打包，构建一个可执行的jar包

- 创建软连接到`/etc/init.d/`目录下

  ```powershell
  sudo ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp
  ```

- 在完成软连接创建之后，我们就可以通过如下命令对`yourapp.jar`应用来控制启动、停止、重启操作了

  ```powershell
  /etc/init.d/yourapp start|stop|restart
  ```











