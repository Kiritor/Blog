---
#Oracle数据库导入、导出imp/exp
&#160; &#160; &#160; &#160;imp/exp命令可以实现oracle数据库的还原、备份、迁移.
&#160; &#160; &#160; &#160;实际的开发中,由于测试和开发"并行",会有开发库、测试库的数据迁移,切换,以及数据库升级等。这些操作都伴随着数据库的导入、导出操作.对于Oracle通过导出、导入来进行数据库的迁移(逻辑)是非常方便的,只要安装了oracle客户端,并建立了连接(通过Net configuration Assistant添加正确的服务命名),你就可以把远端的数据库导出到本地,同样你也可以把dmp文件从本地导入到远端数据库服务器中.利用这个功能,可以构建两个相同的数据库:开发库、测试库,并且快速的实现两个库之间数据的迁移.
#执行环境
&#160; &#160; &#160;可以在SQLPLUS或者直接在CMD命令行中执行.
&#160; &#160; &#160;Tips:在sqlplus环境下执行时需要在前面加'!'号:SQL>!exp ...SQL>!imp
#exp导出命令
&#160; &#160; &#160;exp命令有三种模式(FULL:完全、OWNER:用户、TABLES:表)
##FULL:完全模式
&#160; &#160; &#160;完全模式表示导出整个数据库,必须具备特殊的权限.一个实际的例子:
```SQL
     //导出整个数据库
	 exp demo/demo@demo file=F:\demo.dmp full=y
```
##OWNER:用户模式
&#160; &#160; &#160;用户模式表示导出某个用户下的所有对象及数据(基表、视图、存储过程等).一个例子:
```SQL
    //导出ecisdemo用户下的所有对象
	exp demo/demo@demo owner=user1 file=F:\user1.dmp 
```
&#160; &#160; &#160;当然,也可以同时导出多个用户的数据,一个例子:
```SQL
    //导出user1、user2用户下的对象
    exp demo/demo@demo file=F:\user.dmp owner(user1,user2)
```
##TABLES:表模式
&#160; &#160; &#160;表模式处于用户模式级别下的,用法也比较的灵活,一些例子:
```SQL
	//导出某个用户下的表table1、table2
	exp demo/demo@demo owner=user1 tables(table1,table2) file=F:\user1.dmp 
	//数据过滤
	//导出数据库中标table1中的字段name已"LCore"开头的数据导出
	exp demo/demo@demo owner=user1 tables(table1) query=\"where name like'LCore%'\" file=F:\user1.dmp 
```
##压缩
&#160; &#160; &#160;也可以对导出的数据进行压缩,在上面的命令加上compress=y即可
#imp导入命令
&#160; &#160; &#160;与exp命令相对的,imp也有三种模式:
##FULL:完全模式
&#160; &#160; &#160;完全模式下的命令也比较简单,例子:
```SQL
     imp demo/demo@demo file=F:\demo.dmp full=y
```
##OWNER:用户模式
&#160; &#160; &#160;imp的用户模式导入必须指定FROMUSER:源用户,TOUSER:目标用户这样才能导入数据,一个例子:
```SQL
     将用户fuser的对象导入到用户tuser
     imp demo/demo@demo fromuser=fuser touser=tuser file=F:\demo.dmp
```
&#160; &#160; &#160;Tips:需要注意的是如果是相同用户的话,就没有必要指定FROMUSER和TOUSER参数了.
##TABLES:表模式
&#160; &#160; &#160;表模式的导入也比较简单,一个例子:
```SQL
     //表table1导入
     imp demo/demo@demo tables=(table1) file=F:\demo.dmp
```
&#160; &#160; &#160;通过上面的例子及命令,实际导入、导出已经够了.
#忽略错误
&#160; &#160; &#160;在导入的过程中可能存在一些错误,例如已经存在该表了,可以使用ignore=y来忽略创建错误.
#日志
&#160; &#160; &#160;有时候,需要对导入、导出操作做日志,也很简单,可以使用log=fileName即可.
#谈谈备份
&#160; &#160; &#160;Oracle数据库有两种备份方式:物理备份、逻辑备份.
&#160; &#160; &#160;<B>物理备份</B>:实现数据库的完整恢复,但是数据库必须运行在归档模式下(业务数据库在非归档模式下运行),且需要极大的外部存储设备.
&#160; &#160; &#160;<B>逻辑备份</B>:不需要数据库运行在归档模式下,不但备份简单,而且可以不需要外部存储设备.一般使用此种备份方式.
&#160; &#160; &#160;imp、exp命令即是实现逻辑备份的命令,根据imp、exp命令的不同模式,逻辑备份相应的分为完全备份、用户备份、表备份.