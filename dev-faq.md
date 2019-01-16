layout: post-noTOC
title: Dev-faq
date: 2018-11-15 10:13:17
tags: [spring,java]
categories: work
top: truehe
---
## 前言
突然想把工作中和学习中遇到的开发上的疑难问题记录一下。不仅仅限于代码层面，可以是一些开发技巧，新的思维、研发工具，甚至是一些TODO。
## SSH工具
虽然Mac的终端可以使用SSH命令远程登录服务器，可是每次都需要输入命令挺麻烦的，而且服务器也多不便于管理。推荐一款跨平台的SSH工具**Termius**

## Autowired注解
与 **@Resource** 做个简单对比:
* @Resource属于JAVAEE,@Autowired属于Spring
* @Autowired按类型装配,可配合@Qualifier按名称装配(目前Spring4泛型注入使用该注解,泛型注入还能根据泛型选择)
* @Resource默认按名称进行装配，在按类型装配，如果指定了name属性则只按名称装配

由于 **@Resource**属于JAVAEE,耦合度较低，一般情况下推荐使用
<!--more-->

构建springboot+mybatis通用maper、通用service时，使用 **@Resource** 注解报错如下:
```java
No qualifying bean of type 'tk.mybatis.mapper.common.Mapper<?>' available: expected single matching bean but found 2: roleMapper,userMapper
```
原因在于多个mapper继承了通用mapper, 又使用了泛型，**@Resource**注解无法装配,采用 **@Autowired**解决。

## Dash
开发者必备的API文档聚合工具,随查随用,简单快捷，学会先查API,再Google/baidu 