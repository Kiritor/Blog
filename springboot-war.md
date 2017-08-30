title: Spring Boot(二):war包形式发布
layout: post-noTOC
date: 2017-04-05 10:35:38
tags: springboot
categories: springboot
---

在上一篇入门文章中,也提及了Spring Boot可以发布成war包运行在外包的tomcat中.具体做法如下.

1. 在POM.XML文件中将<packaging>jar</packaging>改为<packaging>war</packaging>


2. 添加依赖.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```
<!--more-->
将tomcat的范围改为provided,编译时需要.

3.接下来添加ServletInitializer类,内容如下:
```java
package com.kiritor.springboot02;

import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```
其中Application为标注有@SpringBootApplication注解的主启动类.
4. 需要注意的问题是,如果配置了server.context-path=/spring-boot配置项,则该配置项的值应该和war包名字一致.