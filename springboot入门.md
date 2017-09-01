title: Spring Boot(一):快速入门
layout: post-noTOC
date: 2017-04-02 11:35:38
tags: springboot
categories: springboot
---
## 简介
Spring Boot是由Pivotal团队提供的全新框架,其设计目的是用来简化新Spring应用的初始搭建及开发过程。JavaEE开发的小伙伴肯定听说过"约定优于配置"、"契约式编程",就是说系统、类库、框架应该假定合理的默认配置，而非要求提供不必要的配置,简化配置项减少跨平台部署容易出现的问题。基于这些问题Spring Boot应运而生，更简单快捷的构建Spring应用。

## Spring Boot优点
Spring Boot让Spring应用简单、快速、轻量化!传统的Spring应用搭建一个基础的spring web项目需要做些什么呢?

>  1) 配置web.xml,加载spring和spring mvc.
>  2) 配置数据库连接、配置spring事务等.
>  3) 配置文件的加载、读取.
>  4) ......

之后的部署,打包十分繁琐.虽然基于"约定优于配置"Spring能够减少了很多的配置项,但是仍然比较臃肿,每个项目都需要这么折腾一遍!但是如果使用微服务Spring Boot框架,只需少量配置即可.
<!--more-->
Spring Boot的主要优点

> 1) 为所有Spring开发者更快的入门.
> 2) 开箱即用(out of the box),提供各种默认配置来简化项目配置.
> 3) 内嵌式容易简化Web项目.
> 4) 没有冗余的代码生成和XML配置的要求.

## 快速入门
完成Spring Boot基础项目的构建,实现一个简单的Http请求处理,对Spring Boot有一个初步的了解,并体验其结构简单、开发快速的特性.

### 系统要求

>  ● JDK 1.7及以上
>  ● 开发工具: IDEA

### Maven构建项目
Spring Boot项目构建方法很多,可以通过官网[http://start.spring.io/](http://start.spring.io/)构建,也可以通过SPRING INITIALIZR插件构建(IDEA社区版不支持).这里笔者通过构建空的maven项目,一步步完成Spring Boot基础工程的搭建.
#### 新建项目
在IDEA选择File->New->Project建立一个项目,考虑到是一个持续的过程，建立一个freestyle structure空项目，之后在File->New->Module建立一个空的Maven Module.
习惯使用Eclipse者初次使用IDEA的注意区分IDEA中的Project与Module的区别.

#### pom.xml
建立好之后,编写pom.xml文件,具体内容如下:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.kiritor.springboot01</groupId>
    <artifactId>springboot01</artifactId>
    <version>1.0</version>
    <!-- -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.3.RELEASE</version>
        <relativePath />
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- web模块 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--测试模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <!--maven 编译插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
#### 项目结构介绍
如下图所示,Spring Boot的基础结构共三个文件:
![空Module](http://o7q5y55yj.bkt.clouddn.com/springBoot%E7%9B%AE%E5%BD%95.png)

>  src/main/java   源码
>  src/main/resource 配置文件
>  src/test/java 测试代码

另外,Spring Boot建议的package目录结构如下所示:

>  1. Application.java建议放到package根目录下面,主要用于做一些框架配置.
>  2. domain目录主要用于实体(Entity)与数据访问层
>  3. service层主要是业务类代码.
>  4. controller负责页面访问控制.

#### 编写hello服务
创建BaseController,内容如下:
```java
@RestController
public class BaseController {

    @RequestMapping("/")
    public String hello(){
        return "hello Springboot";
    }

}
```
#### 简单配置
1、application.properties
```xml
# 应用访问路径
server.context-path=/springboot01
# 端口号
server.port=8081
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
spring.messages.encoding=UTF-8
```
2、banner.txt
该配置可以修改springboot启动的欢迎信息,比较有意思:
#### 运行
1、从IDE运行.
在IDEA中选择Application直接右键运行如下图所示:
打开浏览器访问[http://localhost:8081/springboot01](http://localhost:8081/springboot01),可以看到页面输出hello Springboot
控制台输出结果如下图:
![控制台日志](http://o7q5y55yj.bkt.clouddn.com/springbootlog.png)
2、jar包运行.
通过maven插件执行install命令创建一个可执行的jar(需要注意的是会额外生成一个XXX.jar.original的文件,这是Spring Boot重新打包前maven创建的原始jar文件)使用命令java -jar运行应用,例如:
```bash
java -jar springboot01-1.0.jar
```
3、Maven插件运行
也可以通过maven插件来运行,定位到工程目录下，使用如下命令:
```bash
mvn spring-boot:run
```
4、war包运行
Spring Boot也可以将web工程打成jar包来运行,具体参考文章.

完整源码地址:[https://github.com/Kiritor/Spring-Boot-Study](https://github.com/Kiritor/Spring-Boot-Study),springboot01
