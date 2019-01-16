layout: post
title: "Oracle-数据库实例、表空间、用户、表之间的关系(转)"
date: 2015-12-23 16:28:19
category: work
tags: [数据库实例,表空间,用户]
---
完整的Oracle数据库通常由两部分组成:**Oracle数据库**和**数据库实例**。

>    1)数据库是一系列物理文件的集合(数据文件,控制文件,联机日志,参数文件等);
>    2)Oracle数据库实例则是一组Oracle后台进程/线程以及在服务器分配的共享内存区;

在启动数据库服务时,实际上实在服务器内存中创建一个Oracle实例(即在服务器内存中分配共享内存的后台内存),然后由这个oracle数据库实例来访问和控制磁盘中的数据文件。Oracle有一个很大的内存块,称为全局区(SGA)
<!--more-->
## 一、数据库、表空间、数据文件
### 1、数据库
数据库是数据集合,oracle是一种关系型的数据库管理系统。
通常情况下我们称的数据库,并不仅仅指的是物理的数据集合,他包含物理数据、数据管理系统。也即物理数据、内存、操作系统进程的组合体。
安装过oracle数据库就知道,我们在安装数据库时会让我们选择启动数据库(即默认的全局数据库):
全局数据库:就是一个数据库的标识,在安装时指定,以后一般不会修改.一旦安装,数据库名就写进了控制文件,数据库表,很多地方都会用到这个数据库名。
启动数据库:也叫全局数据库,是数据库系统的入口,它会内置一些高级权限的用户如SYS,SYSTEM等。我们用这些搞基权限账号登陆就可以在数据库实例中创建表空间，用户，表。
```sql
--查询当前数据库名
select name from v$database
```
### 2、数据库实例
oracle官方描述:实例是访问Oracle数据库所需的一部分计算机内存和辅助处理后台进程,是由进程和这些进程所使用的内存(SGA)所构成的一个集合。
其实就是用来访问和使用数据库的一块进程,只存在于内存中。
我们访问oracle都是通过实例来访问到的,如果这个实例关联了数据库文件是可以访问的，否则就会得到实例名不可用的错误。
实例名指的是用于响应某个数据库操作的数据库管理系统的名称。同时也叫SID.实例名由参数instance_name决定的。
查询当前数据库实例名:
```sql
--查询数据库实例名
select instance_name from v$instance;
```
数据库实例名(instance_name)用于对外部连接。在操作系统中要取得与数据库的联系,必须使用数据库实例名。比如我们做开发,要连接数据库,就得连接数据库实例名:
```SQL
jdbc:oracle:thin:@localhost:1521:orcl（orcl就为数据库实例名）
```
一个数据库可以有多个实例,在做数据库服务集群的时候可以用到。
### 3、表空间
Oracle数据库是通过表空间来存储物理表的,一个数据库实例可以有多个表空间,表空间下可以有多个表。
表空间是数据库的逻辑划分,每个数据库至少有一个表空间(SYSTEM表空间)。为了便于管理和提供运行效率,可以使用一些附加表空间来划分用户和应用程序。例如:USER表空间提供一般用户使用，RBS表空间供回滚段使用。一个表空间只能是属于一个数据库。
创建表空间语法：
```SQL
Create TableSpace 表空间名称  
DataFile          表空间数据文件路径  
Size              表空间初始大小  
Autoextend on
```
如:
```SQL
create tablespace db_test  
datafile 'D:\oracle\product\10.2.0\userdata\db_test.dbf'  
size 50m  
autoextend on;
```
查看已经创建好的表空间:
```SQL
select default_tablespace, temporary_tablespace, d.username  
from dba_users d
```
### 4、用户
Oracle数据库建好之后，要想在数据库里建表,必须先为数据库建立用户,并未用户指定表空间(一般指定和表空间一样的名字)
创建新用户:
```SQL
CREATE USER          用户名  
IDENTIFIED BY        密码  
DEFAULT TABLESPACE   表空间(默认USERS)  
TEMPORARY TABLESPACE 临时表空间(默认TEMP)
```
如:
```SQL
CREATE USER utest  
IDENTIFIED BY utestpwd  
DEFAULT TABLESPACE db_test  
TEMPORARY TABLESPACE temp;
```
有了用户,要想使用用户账号管理自己的表空间,还的授权:
```SQL
GRANT CONNECT TO utest;
GRANT RESOURCE TO utest;
GRANT dba TO utest;--dba为最高权限，可以创建数据库，表等。
```
查看数据库用户:
```SQL
select * from dba_users;
```
### 5、表
有了数据库，表空间和用户，就可以用自定义的用户在自己的表空间创建表了。