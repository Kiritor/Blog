title: Shiro-ini配置
date: 2016-04-4 15:30:00
tags: [shiro]
categories: java
---
Shiro的配置十分灵活,支持ini、XML等方式.它的配置是从根对象securityManager开始的.
## 根对象SecrityManager
回顾Shiro的架构图可以知道,shiro是从根对象SecurityManager进行身份验证和授权的,这个对象是线程安全且整个应用只有一个,shiro提供了SecurityUtils让我们绑定它为全局的,方便后续使用.
<!--more-->
Shiro的类都是POJO的,因此都很容易的放到任何的Ioc容器管理,但是和一般的IoC容器(类似Spring)的区别点在于,shiro是从根对象securityMnanager开始导航;其支持的依赖注入:public孔灿构造器对象的创建、setter依赖注入.
## ini配置
int配置文件类似于Java中的properties(key/vale),不过体哦那个了讲key/value分区selection的特性,key是每个分区不重复即可,而不是整个配置文件,比如:
```bash
[main]
#提供了对根对象securityManager及其依赖的配置
securityManager=org.apache.shiro.mgt.DefaultSecurityManager
securityManager.realms=$jdbcRealm
[users]
#提供了对用户/密码及其角色的配置,用户名=密码,角色1,角色2
username=password,role1,role2
[roles]
#提供了角色及权限之间的关系的配置,角色=权限1,权限2
role1=permission1,permission2
[urls]
#用于web,提供了对web url拦截相关的配置,url=拦截器[参数],拦截器
/index.html = anon
/admin/** = authc,roles[admin],perms["permission1"]
```
**[main]部分**
提供了对根对象securityManager及其依赖对象的配置.
**创建对象**
```bash
securityManager=org.apache.shiro.mgt.DefaultSecurityManager
```
其构造器必须是public空惨构造器,通过反射创建相应的实例。
**常量值setter注入**
```bash
dataSource.driverClassName=com.mysql.jdbc.Driver
jdbcRealm.permissionsLookupEnabled=true
```
会自动调用jdbcRealm.setPermissionsLookupEnabled(true),对于这种常量值会自动类型转换.
**对象引用setter注入**
```bash
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator
authenticationStrategy=org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy
authenticator.authenticationStrategy=$authenticationStrategy
securityManager.authenticator=$authenticator
```
会自动通过securityManager.setAuthenticator(authenticator)注入引用依赖.
**嵌套属性setter注入**
```bash
securityManager.authenticator.authenticationStrategy=$authenticationStrategy
```
**byte数组setter注入**
```bash
#base64 byte[]
authenticator.bytes=aGVsbG8=
#hex byte[]
authenticator.bytes=0x68656c6c6f
```
默认需要使用Base64进行编码,也可以使用0x十六进制.
**Array/Set/List setter注入**
```bash
authenticator.array=1,2,3
authenticator.set=$jdbcRealm,$jdbcRealm
```
Map setter注入
```bash
authenticator.map=$jdbcRealm:$jdbcRealm,1:1,key:abc
```
**实例化/注入顺序**
```bash
realm=Realm1
realm=Realm12
```
后面的会覆盖前面的.
**[users]部分**
配置用户名/密码及其角色,格式"用户名=密码,角色1,角色2",角色部分可以省略
```bash
[users]
zhang=123,role1,role2
wang=123
```
密码一般生成摘要/加密存储(md5),后续会使用到.
**[roles]部分**
配置角色及权限之间的关系,格式: "角色=权限1,权限3";如:
```BASH
[roles]
role1=user:create,user:update
role2=*
```
**[urls]部分**
配置url及相应的拦截器之间的关系,格式: "url=拦截器[参数],拦截器[参数]",如:
```bash
[urls]
/admin/** = authc, roles[admin], perms["permission1"]
```
在之后的shiro集成到web的例子中在深入了解.