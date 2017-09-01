title: Spring Boot(三):配置详解
layout: post-noTOC
date: 2017-04-05 11:35:38
tags: springboot
categories: springboot
---
## 写在之前
在前面Spring Boot的入门篇中,可以很直观的感受到使用Spring Boot没有了原来整合Spring应用时繁琐的XML配置内容,替代它的是在pom.xml中引入模块化的Starter POMs,各个模块具有自己的默认配置.在application.properties文件中,只需要设置少量的应用即可开启应用.

在实际的项目中,配置是十分复杂的,存在多个配置环境,例如:
> 开发环境 -> 测试环境 -> 演示环境 ->生产环境

每个环境的配置项总是不同的,而且就算是在开发环境中,不同的开发人员也会有区别,配置文件的读取总是比较伤脑筋.Spring Boot提供了一种优先级配置读取机制很好的解决此问题.
在入门篇中,了解到Spring Boot的配置会从application.properties中读取,实际上这只是Spring Boot配置链中的一环而已.
## 配置优先级
通常应用部署会包括开发、测试和生产等若干个环境,不同环境之间的配置存在覆盖关系.暴力的做法是只有一个配置文件,发版部署的时候在手动去确定某些配置项,容易出错且浪费时间.Spring本身通过引入Environment和概要信息(Profile)API,可以更加灵活的处理不同环境和配置文件的方式.
Spring Boot提供了统一的方式来管理应用的配置,允许开发人员使用属性文件、YAML文件、环境变量和命令行参数来定义优先级不同的配置值.
Spring Boot配置按照优先级从高到低的顺序如下:
> 1. spring-boot-devtools.properties(devtools是active状态的时候)
> 2. @TestPropertySource注解在测试中.
> 3. @SpringBootTest#properties属性注解在测试中.
> 4. 命令行参数.
> 5. 初始化的ServletConfig、ServletContext参数.
> 6. 从java:com/env得到的JNDI属性.
> 7. 通过System.getProperties()获取的系统参数.
> 8. 操作系统环境变量.
> 9. RandomValuePropertySource生成的"random.*"属性.
> 10. 应用jar文件之外的属性文件(通过spring.config.location)参数指定.
> 11. 应用jar文件内部的属性文件.
> 12. 在应用配置Java类(Configuration、PropertySource注解声明的属性文件)
> 13. 通过SpringApplication.setDefaultProperties声明的默认值.

Spring Boot在优先级更高的位置找到了配置,那么它就会无视优先级低的配置.
Spring Boot的配置优先顺序看起来比较复杂,可以知道的是优先级最高的是dev-tools用于实现热部署,下面简单的看看一些配置.
### 命令行参数
命令行参数的好处是在于在某些情况下快速的修改配置参数值,而不需要重新打包和部署应用.例如:
```bash
java -jar springboot01-1.0.jar --server.port=9999
```
部署的时候使用动态指定端口号位9999.SpringApplication类会默认把以"--"开头的命令行参数转化为应用中可以使用的配置参数.等价于在application.properties添加属性server.port=9999.
通过命令行来修改属性值固然提供了不错的便利性,但是这样就能更改应用运行的参数存在安全隐患.可以通过如下代码禁用命令行访问属性的设置.
```java
SpringApplication.setAddCommandLineProperties(false)
```
### 属性配置文件
属性文件是最常见的管理配置属性的方式,Spring Boot提供的SpringApplication类搜索并加载application.properties文件来获取配置属性值(也可以是yml文件,但是.properties文件优先级高).搜索规则如下:
> ● 当前目录的"/config"子目录.
> ● 当前目录.
> ● classpath中的"/config"包.
> ● classpath.

上面的顺序也表示了该位置上包含的属性文件的优先级.优先级按照从高到低的顺序排列.在application.properties中可以配置诸如端口号,数据库链接等信息.例如:
```bash
server.context-path=/springboot01
server.port=8081
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
spring.messages.encoding=UTF-8
```
### 自定义属性与加载
在使用Spring Boot的时候,有时候也需要自定义一些属性,例如我们自定义如下属性(在application.properties中):
```bash
user.userName=Kiritor
user.age=25
user.id=1
```
然后通过@Value("${属性名}")注解来加载对应的自定义属性到javaBean中,具体如下:
```java
@Component
public class User {

    @Value("${user.id}")
    private String id;
    @Value("${user.userName}")
    private String userName;
    @Value("${user.age}")
    private String age;
    //省略getter、setter
}
```
Spring进行JavaBean管理:
```java
    @Resource
    User user;
    @RequestMapping("/")
    public String hello(){
        return user.getId()+":"+user.getUserName()+":"+user.getAge();
    }
```
#### 参数间引用
各个参数之间的属性是可以引用的,例如:
```bash
user.des=${user.userName}的年龄是:${user.age}
```
#### 将配置文件的属性一次性注入到JavaBean中
上面的例子中,通过@Value("${}")方式一个一个注入比较麻烦,Spring Boot提供一次性的注入.
新建一个NewUser类:
```java
@Component
@ConfigurationProperties(prefix = "user")
public class NewUser {
    private String id;
    private String userName;
    private String age;
    //省略getter、setter
```
通过@ConfigurationProperties注解,指定其prefix前缀(例如以user为前缀的属性值会被自动绑定到java类中同名的域上),之后需要在应用类或者application类加上EnableConfigurationProperties注解就可以启用起自动绑定功能,例如:
```java
@RestController
@EnableConfigurationProperties({NewUser.class})
public class NewUserController {
    @Autowired
    NewUser newUser;
    @RequestMapping("/newuser")
    public String hello(){
        return newUser.getId()+":"+newUser.getUserName()+":"+newUser.getAge()+"||"+newUser.getDes();
    }
}
```

### 配置随机值
RandomValuePropertySource可以用来生成测试所需要的各种不同类型的随机值,从而免去了代码生成的麻烦.通过${random.}由RandomValuePropertySource生成.例如:
```xml
user.id=${random.value}
user.userName=${random.value}
user.age=${random.int[1,100]}
```
### 自定义配置文件
有时候我们不愿意把配置都写到一个文件中去(application.properties),而是分散到多个配置文件方便管理,Spring Boot也支持自定义配置文件.
新建一个配置文件test.properties:
```bash
com.test.userName=test
com.test.age=20
```
同样,接下来将该配置文件信息赋予给一个javabean作为例子:
```java
@Component
@PropertySource(value = "classpath:/config/test.properties")
@ConfigurationProperties(prefix = "com.test")
public class Test {
    private String userName;
    private Integer age;
    //省略getter、setter
}
```
之后在应用类或者application类加上EnableConfigurationProperties即可自动绑定:
```java
@RestController
@EnableConfigurationProperties({NewUser.class , Test.class})
```
### CommandLineRunner
项目启动的时候,需要先启动一些初始化的类,比较常见的做法是通过static块.Spring Boot提供了一个CommandLineRunner接口,实现这个接口的类总是会被优先启动,并优先执行run方法.
```bash
public class ApplicationConfigure implements CommandLineRunner {
    @Value("${app.sysName}")
    private String sysName;
    @Override
    public void run(String... strings) throws Exception {
         //预先加载一些类、图片、属性等
    }
}
```
### 多环境配置
实际项目中,通过会存在不同的环境(开发、测试、生成等).其中每个环境的配置是有区别的,例如数据库地址、服务器端口号,如果在为不同环境打包部署的时候都要频繁的修改配置文件的话,那必将是个繁琐且头疼的问题.

对于多环境的配置,各种项目构建工具或者框架的思路基本一致:通过配置不同环境的配置文件,再通过打包命令指定需要打包的内容之后进行区分打包.
以application.properties为例,通过文件名来区分环境application-{profile}.properties.
新建开发环境和测试环境application-dev.properties、application-test-properties
其中application-dev.properties内容如下:
```bash
server.port=10086
```
application-test.properties内容如下:
```bash
server.port=10096
```
application.properties内容如下:
```bash
server.port=8081
#dev
spring.profiles.active=dev
```
通过spring.profiless.active属性来设置哪个具体的配置文件会被加载,其值对应{profile}值.
dev环境启动端口为10086,test环境启动端口为10096,且默认application的端口为8081,在实际发版部署的时候,会根据指定的profile加载覆盖正确的配置属性.
Spring Boot会先加载默认配置文件,然后使用具体指定的profile中的配置去覆盖默认配置.

基于以上尝试,Spring Boot多环境配置步骤如下:
> ●  application.properties中配置通用内容,并设置spring.profiles.active=dev,以开发环境为默认配置
> ●  application-{profile}.properties配置各个环境不同的内容
> ●  通过命令行方式(允许的话)或者修改默认配置文件的pring.profiles.active值激活不同环境的配置

### 指定外部的配置文件
有些系统中,基于安全和机密的考虑,关于一些数据库或者第三方账户等信息,起配置并不会配置在项目中暴露给开发人员,对于这种情况,我们可以在运行程序的时候,通过--spring.config.location参数指定一个外部配置文件.
```bash
java -jar springboot03.jar --spring.config.location=d:\\application.properties
```
其中外部文件的文件名可自行定义.
以上就是Spring Boot的一些相关配置,以及如何配置一些简单的配置项.至于后续hibernate的配置,日志配置在慢慢探索.

