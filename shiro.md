title: Shiro简介
date: 2015-09-18 15:35:01
tags: [shiro]
categories: java
---
## 简介
Apache Shiro是apache的一个强大而灵活的开源安全框架,它干净利落的处理身份认证、授权、企业会话管理和加密.Shiro首要的目标是易于使用和理解,提供一个干净而直观的API,来简化开发人员在使用它们的应用程序安全上的努力.这也是Shiro可能并没有Spring Security使用的人却越来越多的因素之一吧.

Shiro不强制依赖其他第三方框架、容器、或者应用服务器,不仅可以应用在JavaSE环境,也可以用在JavaEE环境.

## Shiro Features
Apache Shiro是一个拥有许多功能的综合性的程序安全框架,帮助我们完成:认证、授权，加密、会话管理、与web集成、缓存等功能.shiro提供的功能如下图:
<!-- more -->
<center>![Shiro Features](http://kiritor.github.io/img/shiro_features.png)</center>
Shiro开发团队把身份验证、授权、会话管理、加密作为核心关注点--应用程序四大基石.
*	**Authentication:** 身份认证/登录,验证用户是不是具有相应的身份;
*   **Authorization:** 访问控制,授权,即是权限验证,验证用户是否具有某项权限;
*   **Session Management:**  会话管理,管理用户特定的会话,适用于JavaSE环境、Web环境;
*   **Cryptography:**  通过加密算法保证数据安全性,同时易于使用;
Shiro同样提供了额外的功能来加强在不同环境所关注的方面;
*   **Web Support:**  Web支持的API能够非常容易的集成到web环境中去;
*   **Caching:**  缓存,提高查询效率;
*   **Concurrency:**  并发特性,支持多线程应用;
*   **Testing:**  提供测试支持;
*   **Run As:**  允许一个用户假设为另一个用户的功能;
*   **Remember me:**  记住我,十分常用的功能,登录之后,下次就不用登录了;
**Shiro提供了这么多的可插拔化的功能,使用时可以根据需要来选择,但是,Shiro不会去维护系统用户,更不会去维护系统的权限规则.针对不同系统,结合自身业务权限设计模型千差万别,Shiro不可能统一维护,也不会提供一套方案.因此对于权限仍然需要我们自行设计/提供,然后通过相应的接口注入给Shiro.**
## Shiro Architecture
Shiro的架构主要有3个主要的概念:Subject，SecurityManager,Realms.
### High-Level Overview
下面的关系图是关于上面上个组件是如何交互的高级概述:
<center>![shiro_high](http://kiritor.github.io/img/shiro_high.png)</center>
可以看到,直接交互对象是Subject,其每个对象的含义:
**Subject:** 主体,代表了当前"用户",这是一个抽象概念,代表了和应用交互的任何东西,所有的Subject都绑定到SecurityManager,与Subject的交互都委托给SecurityManager,SecurityManager是实际的执行者.
**SecurityManager:** 安全管理器:即所有的与安全有关的操作都会与SecurityManager交互,他是Shiro架构的心脏,控制着所有的Subject协调内部的安全组件之间的交互.
**Realm:** 域,shiro通过Realm获取安全数据(用户、角色、权限规则等),例如SecurityMananger要验证身份,需要从Realm得到用户的相应信息来确定用户信息是否合法.可以把Realm看做是安全数据源DataSource.从这个意义上来说,Realm本质上是一个特定安全的Dao,它封装了数据源的连接详细信息.shiro提供了立即可用的Realms来连接一些安全数据源(LDAP,数据库,文本配置源ini)
对于我们来说,一个Shiro应用:
>   1、应用代码通过Subject来进行认证和授权,Subject委托给SecurityManager;
>   2、SecurityMananger通过Realm得到合法的用户及权限;

上面说到了Shiro不维护用户/权限,而是通过Realm让开发人员自己注入.

### Shiro Architecture
下图展示的是Shiro内部详细的架构图:
<center>![shiro_detail](http://kiritor.github.io/img/shiro_detail.png)</center>
*    **Subject:*** 主体,当前与软件进行交互的实体(用户,第三方服务)等.
*    **SecurityManager:** SecurityManager是Shiro的心脏,协调其管理的组件,确保他们能够一起顺利的工作,执行每个Suject的安全操作.
*    **Athenticator:** 认证器,负责主题认证,这是一个可扩展点,可以自定义实现;其需要认证策略(Authentication Strategy),及什么时候用户认证通过.
*    **Authrizer:** 访问控制器,决定主体是否有权限进行相应的操作,控制用户能访问应用程序的哪些功能.
*    **Realm:** 可以有一个或者多个实现, 安全数据源,用于获取安全实体;可以是JDBC实现,也可以是LDAP实现等.上面也说道了Shiro不维护用户/权限,因此我们一般应用中都需要实现自己的Realm;
*    **SessionManager:** 管理用户session生命周期,shiro可以在任何环境下本地化管理用户的session,及时没有可用的Web/servlet容器,他将会使用内置的企业级会话管理来提供同样的编程体验.
*    **SessionDao:** SessionDao完成session的持久化操作(CRUD),例如放入数据库中,memcached中;另外SessionDao中可以使用缓存,已提高性能.
*    **CacheManger:** 缓存管理,管理用户,角色、权限等缓存,这类数据一般较少改变,使用缓存可以提升性能.
*    **Cryptography** 加密模块,安全框架的自然补充,提供了一些常见的加密组件用于如密码的加密/解密.
到此为止,对于Shiro框架有了一个大致的了解,接下来也就是跟着资料文档,一步步去实践其各个组件功能了.这也是近期的目标.
