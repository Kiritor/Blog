title: Oracle dblink实践
date: 2015-04-14 17:22:49
tags: [oracle,dblink,hibernate]
categories: work
---
## 前言
&nbsp;&nbsp;&nbsp;&nbsp;项目开发中,涉及到不同模块之间的数据流转,但是模块间的底层数据又不在同一个数据库中,要实现不同模块间的数据交互方法其实很多。比较常见的两种方式便是webService和dblink。

&nbsp;&nbsp;&nbsp;&nbsp;webService方式即是模块之间各自提供数据流入接口和流出接口,这种方式需要开发,而且由于业务变化很容易造成接口的调整,好处是各业务模块的底层数据库是完全耦合的。在大型系统(模块多且之间交互复杂)不利。

&nbsp;&nbsp;&nbsp;&nbsp;dblink的方式是通过创建dblink到远程数据库,执行远程程序,这样一来,模块间的数据流转就会变得非常简单,各自模块无需提供数据接口。但是有一个问题是,模块底层数据库之间不是耦合的,在系统实际上线之前我们要规范好各个模块数据库之间的link关系,之后按照规范,部署数据库实例,前期的准备工作比较麻烦。
<!--more-->

&nbsp;&nbsp;&nbsp;&nbsp;经过一些权衡,以及实际项目开发的情况,选择了以dblink的方式来解决各模块之间数据流转的问题
## dblink概述
&nbsp;&nbsp;&nbsp;&nbsp;dblink定义一个数据库到另一个数据库路径的对象,它允许你查询远程表及执行远程程序,在任何分布式的生产环境中,dblink都是必要的(结合物化视图),而且dblink是单向的连接。
dblink存在于多个数据库之间,因此在创建dblink之前需要保证本地库和远程库的网络连接是正确的,TNS Ping要能成功,而且该用户在远程数据库上有相应的访问权限。

对于dblink就不深入介绍了,比较有参考意义的一篇文章:[Oracle dblink详解](http://czmmiao.iteye.com/blog/1236562)
## dblink的创建
创建dblink有两种方式:基于TNS Name和不基于TNS Name
首先是基于TNS Name,语法如下:
```SQL
      CREATE SHARED PUBLIC database link GNIS
      AUTHENTICATED BY USERNAME IDENTIFIED BY PASSWORD
      USING ‘TNSNAME’;
```
其次,不使用TNS Name进行创建:
```SQL
	CREATE database link link_name
	CONNECT TO user IDENTIFIED BY screct
	USING '(DESCRIPTION =
		(ADDRESS_LIST =
			(ADDRESS = (PROTOCOL = TCP)(HOST = sales.company.com)(PORT = 1521))
		)
		(CONNECT_DATA =
			(SERVICE_NAME = sales)
		)
	)';

```
## dblink的使用
dblink的使用方式很简单,一个简单的例子如下:
```SQL
SELECT * FROM table_name@database link;
--不想让人知道dblink的名字的时候
--可以使用oracle的synonym(同义词)进行包装下
CREATE SYNONYM table_name FOR table_name@database link;
SELECT * FROM table_name;
-- 或者，也可以建立一个视图来封装
CREATE VIEW table_name AS SELECT * FROM  table_name@database link;
```
对于dblink的更详细的一些操作,可以参考上述链接(大牛的文章),这里就不过多做介绍了。
## Hibernate使用dblink
&nbsp;&nbsp;&nbsp;&nbsp;了解了dblink理论以及dblink的使用,接下来的重点就是如何将其用于生产环境中了,这里已我自己实际中遇到的例子为例.
首先是创建dblink:
```SQL
    CREATE PUBLIC database link ecistest_dblink
    AUTHENTICATED BY ecis IDENTIFIED BY ecis
    USING 'JULI'
```
其次为了方便hibernate的操作我们使用oracle的同义词SYNONYM给要查询的对象创建一个封装名(能够用SYNONY是因为hibernate是支持同义词映射的),方法如下:
```SQL
--CREATE SYNONYM SYN_TABLE_NAME FOR MM_MATERIALCATEGORY@ecistest_dblink;
--通过别称使用
select count(*) from syn_table_name;
```
之后在hibernate映射文件或者entity类中(注解)使用别名进行底层数据表的映射了,以注解为例:
```JAVA
@Entity
@Inheritance( strategy = InheritanceType.TABLE_PER_CLASS )
@Table( name = "syn_table_name" )
```
接下来,我们就可以使用hibernate通过dblink的方式操作远程数据库了,不过实际运行会报错,错误信息如下:
```bash
ORA-24777 不可使用不可移植的数据库链路
```
通过谷歌,发现是dblink的创建有问题,需要创建一个共享的数据库连接,因此删除创建好的dblink,重新创建shared方式的dblink:
```SQL
DROP database link ecistest_dblink;
CREATE SHARED database link ecistest_dblink
AUTHENTICATED BY ecis IDENTIFIED BY ecis
USING 'JULI'
```
shared dblink需要指定dblink authentication(该用户必须能在远程数据库创建session,且具有数据库相关操作权限)
接下来,运行程序很可能出现如下的错误:
```BASH
ORA-02020 : 过多的数据库链接在使用中
```
出现该错误的原因是shared方式的dblink数据库会限制到远程数据库的session数量,这样以避免过多的连接对远程数据库造成太大的压力,而且在使用shared dblink的时候,到database link的连接会在连接以后与本地连接断开,为防止为授权的session还需要指定link_authentication,关于这方面更多的资料,参考上述连接.

出现上述错误,我们可以通过扩增dblink连接数来解决,方法如下:
1、首先看一下连接数:
```SQL
show parameter open_links;
--笔者修改过后的为
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
open_links                           integer     50
open_links_per_instance              integer     50
open_links:每个session最多允许的dblink数量
open_links_per_instance：指每个实例最多允许的dblink个数
```
2、修改连接数
```SQL
 alter system set open_links=50 scope=spfile;
 alter system set open_links_per_instance=50 scope=spfile;
```
3、重新启动数据库
修改之后需要重新启动数据库该设置才能够生效。
之后在hibernate中就可以正常的使用dblink了。
## 问题补充1
&nbsp;&nbsp;&nbsp;&nbsp在实际使用的过程中,还出现了一个问题,那就是由于oracle版本不一致导致的,一个为oracle10g一个为oracle11g。错误信息如下:
```bash
ORA-01017: 用户名/口令无效; 登录被拒绝
ORA-02063: 紧接着 2 lines (起自 POSTGRESQL)
```
通过查询得知这是由于oracle11G用户名、密码大小写敏感的原因,而且oracle10的网关日志中显示用户名、密码最后都是以大写的形式呈现的。因此创建shared dblink的方式需要修改成如下形式:
```SQL
CREATE SHARED PUBLIC database link mccprd_dblink
CONNECT TO "cpmtest" IDENTIFIED BY "cpmtest"
AUTHENTICATED BY "cpmtest" IDENTIFIED BY "cpmtest"
USING '(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = 10.73.1.141)(PORT = 1521))
)
(CONNECT_DATA =
(SERVICE_NAME = MCCPRD)
)
)';
```
## 问题补充2
&nbsp;&nbsp;&nbsp;&nbsp在实际使用dblink时发现了一个问题:
```bash
ORA--22992:无法使用远程表选择的LOB定位器，database link
```
出现这个问题是因为跨库连接查询的数据表存在CLOB、或则BLOB类型的字段。需要注意:
```bash
1、不能使用select * from x
2、不能将blob、clob类型的字段出现在脚本中。
```
解决方案有两种,第一种:select X.name的形式,去掉blob类型的字段,这种方法不是非常恰当(除非不需要blob类型的字段)
第二种方案:使用零时表,之后把远程的含BLOB字段的表导入到零时表中,再导入到表
```SQL
create global temporary table demo_temp as select * from demo;
insert into demo_temp select * from demo@D_LINK;
insert into demo select * from demo_temp;
commit;   
```