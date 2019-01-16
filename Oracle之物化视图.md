title: Oracle之物化视图
date: 2015-04-14 15:27:41
tags: [物化视图,oracle]
categories: work
---
## 问题描述
&nbsp;&nbsp;&nbsp;&nbsp;项目中,物料的摘要字段是通过视图拼接各个基础字段形成的,单条查询并不会存在性能问题。但是考虑到物料的结构化,对摘要进行搜索的时候,如果物料库的大小以量级的大小增加,那么性能将是一个严重的问题。单纯的视图优化(索引等)并不能根本上解决该问题。经过一些思考,决定采用物化视图的方式来解决。
## 物化视图概述
&nbsp;&nbsp;&nbsp;&nbsp;物化视图(material view)是相对于普通视图而言的,普通的视图是虚拟表,本质上是DBMS转换为对视图SQL语句的查询,性能上没有好处。物化视图可以看成是一种特殊的物理表,他包括一个查询结果的数据库对象,可以是远程数据库的本地副本,也可以是基于数据基本求和的汇总表。物化视图存储基于远程的数据(本地可以以),也被称为快照。
<!--more-->

&nbsp;&nbsp;&nbsp;&nbsp;物化视图可以查询表,视图和其他物化视图.
## 物化视图分类
&nbsp;&nbsp;&nbsp;&nbsp;物化视图一般情况下有三种:包含聚集的物化视图、只包含连接的物化视图、嵌套的物化视图。不过这种分类的方式并不是从其特点上来进行分类的,不利于理解。下面看另几种分类方式: 
```bash
         1、刷新方式:  FAST/COMPLETE/FORCE

         2、刷新时间:  ON DEMAND/ON COMMIT

         3、是否可更新: UPDATABLE/READ ONLY

         4、是否支持查询重写: ENABLE QUERY REWRITE/DISABLEQUERY REWRITE 
```
&nbsp;&nbsp;&nbsp;&nbsp;默认的情况下,Oracle的刷新模式为FORCE何DEMAND 
## 创建方式
物化视图的创建有两种方式BUILD IMMEDIATE 和 BUILD DEFERRED两种。

BUILD IMMEDIATE是在创建视图的时候就生成数据.

BUILD DEFERRED则是在创建的时候不生成数据,以后根据需要生成数据,Oracle默认是按照BUILD IMMEDIATE方式创建的。这个仅仅是说了视图的创建方式,后续介绍完视图的相关特点之后,再介绍如何完整的创建一个物化视图。 

## 刷新模式
按刷新模式来区分物化视图类别是比较常规合理的方式。这里的刷新模式包含两个方面的内容:刷新方式和刷新时间,以刷新时间为主。

前面也说了,物化视图的刷新方式有两种ON COMMIT 和 ON DEMAND

ON COMMIT指的是当基表一旦有了commit(事务提交),就会立刻更新物化视图,使得数据和物化视图一致,但是值得注意的是对于基表来说,平常的commit操作,在设置物化视图刷新方式为ON COMMIT之后速度会大大降低,实际开发中基本不纳入考虑范围。

ON DEMAND顾名思义,仅在该物化视图需要刷新的时候才进行刷新,我们可以手工的通过DBMS.MVIEW.REFRESH的方法进行刷新(编写存储过程),也可以通过JOB定时任务进行刷新,甚至可以编写脚本定时更新物化视图,保证其数据和基本数据保持一致。

前面提及了物化视图的三种刷新方式,COMPLETE、FAST、FORCE。这其实是物化视图在刷新时生成数据的方式。

   1. 完全刷新 (COMPLETE): 会先删除物化视图中的数据,在重新生成数据。

   2. 快速刷新 (FAST): 采用增量刷新的方式,只将上次刷新以后对基表进行的所有操作刷新到物化视图中去。FAST刷新方式,必须建立基于主表的视图日志。

   3. FROCE方式: 自动判断是否满足增量刷新方式,满足则进行增量刷新,反之进行完全刷新。 
   
## 查询重写

包括ENBLE QUERY REWRITE 和DISABLE QUERY REWRITE,指出物化视图是否支持查询重写。查询重写指的是当对物化视图的基表进行查询时,Oracle会自动判断能否通过查询查询物化视图得到数据,如果可以则避免了聚集和连接操作,直接从物化视图中查询,效率会块很多。默认情况不支持查询重写。 

## 创建物化视图
了解上上述关于物化视图的一些知识,下面来看看如何创建物化视图吧。工具选择PL/SQL。

物化视图创建参数: 
```bash
		BUILD -- 创建方式
		REFRESH -- 刷新(获取数据房还是)
		ON(ON DEMAND)--刷新方式
		START WITH-- 通知数据库完成从主表到本地表第一次复制的时间
		NEXT-- 说明刷新的时间间隔
		ENBALE QUERY REWRITE -- 是否支持查询重写
```
简单的例子:
```SQL
		--创建基表
		CREATE TABLE TEST(
			 order_id varchar(20),
			 job_id varchar(20) PRIMARY KEY
		)

		--创建基表日志
		CREATE MATERIALIZED VIEW LOG ON TEST WITH ROWID,SEQUENCE(job_id,order_id) INCLUDING NEW VALUES;

		--创建物化视图
		CREATE MATERIALIZED VIEW MV_TEST
		BUILD IMMEDIATE--默认,创建时生成数据
		REFRESH FAST--FAST必须创建基表日志
		ON DEMAND
		START WITH SYSDATE --第一次刷新时间
		NEXT SYSDATE + 1 --以后每一天刷新一次
		AS 
		SELECT * FROM TEST
```
实例2:
```SQL
		CREATE MATERIALIZED VIEW MM_TOGETHER_MATERIALIZED
		REFRESH COMPLETE ON DEMAND
		START WITH TO_DATE('25-03-2015 11:12:09', 'DD-MM-YYYY HH24:MI:SS') NEXT SYSDATE + 5 
		AS
		SELECT
		  material.id id,
		  REPLACE (
			WMSYS.WM_CONCAT (av. VALUE || U .unitname),
			',',
			'；'
		  )  || ' ' || material.materialname || ' ' || material.materialcode || ' ' || material.remark || ' ' || CATEGROY.categoryname together
		FROM
		  mm_attrvalue av,
		  mm_unit U,
		  MM_MATERIALMNG material,
		  MM_MATERIALCATEGORY categroy
		WHERE
		  material. ID = av.materialid(+)
		AND av.unitid = U . ID (+)
		AND CATEGROY.id = MATERIAL.CATEGORYID
		GROUP BY
		(
		   MATERIAL.ID,
		   material.materialname,
		   material.materialcode,
		   material.remark,
		   CATEGROY.categoryname
		);
```
## 删除物化视图
删除物化视图及物化视图日志的时候传统的Drop语句不起作用,需要使用如下语句: 
```SQL
		DROP MATERIALIZED VIEW LOG ON GG_ZLX_ZHU@TOCPEES;
		DROP MATERIALIZED VIEW GG_ZLX_ZHU;
```
## 存储过程刷新
上面提及了可以通过DBMS.MVIEW.REFRESH来刷新物化视图。而且根据一些业务场景的需要,可能不定时刷新,所以不能是JOB,而且如果数量多也不能一个个刷新。编写的存储过程如下: 
```SQL
		CREATE OR REPLACE PROCEDURE P_MVIEW_REFRESH AS

		BEGIN 

		DBMS_MVIEW.refresh('MM_TOGETHER_MATERIALIZED,MM_MMDESCRIPTION_MATERIALIZED','cc');

		END;
```
第一个参数为物化视图名称,多个以","分隔,第二个参数为每个视图对应的刷新方式(f:增量刷新,c:完全刷新,?:强制刷新)

之后即可在命令窗口中通过 exec P_MVIEW_REFRESH即可执行该存储过程。 