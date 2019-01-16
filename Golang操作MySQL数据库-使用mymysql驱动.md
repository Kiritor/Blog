title: "Golang操作MySQL数据库,使用mymysql驱动"
date: 2015-04-09 09:34:07
tags: [mymysql]
categories: golang
---
玩golang也有几周了,了解了基本的语法之后,做了一个简单的webdemo(反射、数据库操作、路由等)，对于系统来说很多都离不开数据库。此篇文章就是对golang如何进行数据库(mysql)操作进行一下尝试,使用的数据库驱动为mymysql
## MyMySQL驱动
MyMySQL的原作者是波兰的[ziutek](https://github.com/ziutek/mymysql),他根据mysql的协议标准使用go语言实现了mymysql包,该包可以用在mysql4.1或更高的版本上,且在5.x系列版本上经过了项目的实际验证。
<!--more-->
## 安装
安装mymysql包必须要先安装如下包:
```bash
go get github.com/ziutek/mymysql/thrsafe
go get github.com/ziutek/mymysql/autorc
go get github.com/ziutek/mymysql/godrv
...
```
```bash
go get github.com/ziutek/mymysql
```
打开gopath目录下的mymysql包可以看见如下的几部分:
####1、mysql:go实现的mysql客户端(没有额外的依赖模块)
####2、native:线程不安全的引擎
####3、thrsafe:线程安全的引擎
####4、autorc：自动重新连接接口
####5、godrv:go提供的database/sql的实现
OK,也就是说,使用mymysql驱动,我们可以灵活多样的操作mysql数据库了,下面看一些实际的例子。
## 实例
上面已经说了,mymysql提供了5个子包,我们可以选择灵活的搭配来操作数据库,例如线程安全与非线程安全(到底如何选择这点是基于业务是否要求多线程,线程安全必然更加耗费性能)。
## mysql+thrsafe/native
首先尝试一下mysql子包的用法,通过查阅文档,编写的代码如下:
```bash
package main

import (
    "fmt"
	//"github.com/ziutek/mymysql/autorc"  //自动连接
    "github.com/ziutek/mymysql/mysql"
    _ "github.com/ziutek/mymysql/native" // 线程不安全
    // _ "github.com/ziutek/mymysql/thrsafe" // 线程安全
)

func main() {
    //得到一个连接
    db := mysql.New("tcp", "", "127.0.0.1:3306", "root", "root", "webdemo")
    //建立连接
    err := db.Connect()
    if err != nil {
        panic(err)
    }
    //查询
    rows, result, err := db.Query("select * from user")
    if err != nil {
        panic(err)
    }

    for _, row := range rows {
        fmt.Println(row.Str(0)+":"+row.Str(1)+":"+row.Str(2))  //打印
    }
	fmt.Println(result.AffectedRows())  //影响行数
	fmt.Println(result.Message())
}
```
上述代码只是一个非常简单的操作,详细的细节处理(预处理,元数据),可以查阅文档。
## database/sql+godrv
mymysql也提供了golang原生的操作mysql的实现,使用这种方式的兼容性更加可靠,也是推荐的做法
```bash
package main
import (
    "fmt"
    "database/sql"
    "github.com/ziutek/mymysql/godrv"
)
// 用户结构
type User struct {
    uid int
    username string
    password string
}
func main() {
    // 设置连接编码
    godrv.Register("SET NAMES utf8")
    // 连接数据库
    db, err := sql.Open("mymysql", "tcp:127.0.0.1:3306*webdemo/root/root")
    if err != nil {
        panic(err)
    }
    defer db.Close()
    // 插入数据
    stmt, err := db.Prepare("insert into user values(null, ?, ?)")
    if err != nil {
        panic(err)
    }
    defer stmt.Close()
    // sql参数
    result, err := stmt.Exec("LCore", "LCore")
    if err != nil {
        panic(err)
    }
    // 获取影响的行数
    affect, err := result.RowsAffected()
    if err != nil {
        panic(err)
    }
    fmt.Printf("%d\n", affect)
    // 获取自增id
    id, err := result.LastInsertId()
    if err != nil {
        panic(err)
    }
    fmt.Printf("%d\n", id)
    // 查询数据
    rows, err := db.Query("select * from user")
    if err != nil {
        panic(err)
    }
    defer rows.Close()
    // 获取的用户
    users := []User{}
    // 读取数据
    for rows.Next() {
        user := User{}
        err := rows.Scan(&user.uid, &user.username, &user.password)
        if nil != err {
            panic(err)
        }
        users = append(users, user)
    }
    // 显示用户信息
    for _, user := range users {
        fmt.Printf("%d, %s, %s\n", user.uid, user.username, user.password)
    }
}
```
## 事务处理
mymysql驱动同样支持事务处理,一个简单的例子(基于mysql子包):
```bash
package main

import (
    "fmt"
    "github.com/ziutek/mymysql/mysql"
    //_ "github.com/ziutek/mymysql/native" // 线程不安全
     _ "github.com/ziutek/mymysql/thrsafe" // 线程安全
)

func main() {
    db := mysql.New("tcp", "", "127.0.0.1:3306", "root", "root", "webdemo")

    err := db.Connect()
    if err != nil {
        panic(err)
    }

    rows, result, err := db.Query("select * from user")
    if err != nil {
        panic(err)
    }

    for _, row := range rows {
        fmt.Println(row.Str(0)+":"+row.Str(1)+":"+row.Str(2))
    }
	fmt.Println(result.AffectedRows())
	fmt.Println(result.Message())
	ins,err:=db.Prepare("insert into user values(?,?,?)")
	if err!=nil {
		fmt.Println(err)
	}
	//开启事务,db处于lock状态,只有当事务提交或者回滚之后才会解锁
	tr,_:=db.Begin()
	//处于事务且线程安全
	/**
      start方法属于db,因此下面两条插入语句都不会执行
    */
	go func(){
		tr.Start("insert user valies(50,'LCore','LCore')")
	}()
	tr.Start("insert user valies(70,'LCore','LCore')")
	tr.Do(ins).Run(60,"LCore","LCore")
	tr.Commit()   //事务提交
}
```
## Type Mapping
我们可以使用格式化的方式嵌入到查询sql中去,例如:
```bash
rows, res, err := db.Query("select * from X where id > %d", id)
```
mymysql的查询结果对应在[]byte类型中,因此需要自己转换为对应的类型
```bash
fmt.Println(string(rows[0][1].([]byte)))
```
或者使用Str函数
```bash
fmt.Println(rows[0].Str(1))
fmt.Pritnln(rows[0].int(1))
```
以上的例子都是返回结果矩阵中的第0行,第一列(下标0开始),但是一般情况下,我们习惯于通过列名进行访问.
```bash
name := res.Map("name")
fmt.Print(rows[0].Str(name))
```
MySQL服务器映射/转换特定MySQL存储类型。
```bash
         string  -->  MYSQL_TYPE_STRING
         []byte  -->  MYSQL_TYPE_VAR_STRING
    int8, uint8  -->  MYSQL_TYPE_TINY
  int16, uint16  -->  MYSQL_TYPE_SHORT
  int32, uint32  -->  MYSQL_TYPE_LONG
  int64, uint64  -->  MYSQL_TYPE_LONGLONG
      int, uint  -->  protocol integer type which match size of int
           bool  -->  MYSQL_TYPE_TINY
        float32  -->  MYSQL_TYPE_FLOAT
        float64  -->  MYSQL_TYPE_DOUBLE
      time.Time  -->  MYSQL_TYPE_DATETIME
mysql.Timestamp  -->  MYSQL_TYPE_TIMESTAMP
     mysql.Date  -->  MYSQL_TYPE_DATE
  time.Duration  -->  MYSQL_TYPE_TIME
     mysql.Blob  -->  MYSQL_TYPE_BLOB
            nil  -->  MYSQL_TYPE_NULL
```
收到结果MySQL存储类型被映射到/ mymysql类型如下:
```bash
                             TINYINT  -->  int8
                    UNSIGNED TINYINT  -->  uint8
                            SMALLINT  -->  int16
                   UNSIGNED SMALLINT  -->  uint16
                      MEDIUMINT, INT  -->  int32
    UNSIGNED MEDIUMINT, UNSIGNED INT  -->  uint32
                              BIGINT  -->  int64
                     UNSIGNED BIGINT  -->  uint64
                               FLOAT  -->  float32
                              DOUBLE  -->  float64
                             DECIMAL  -->  float64
                 TIMESTAMP, DATETIME  -->  time.Time
                                DATE  -->  mysql.Date
                                TIME  -->  time.Duration
                                YEAR  -->  int16
    CHAR, VARCHAR, BINARY, VARBINARY  -->  []byte
 TEXT, TINYTEXT, MEDIUMTEXT, LONGTEX  -->  []byte
BLOB, TINYBLOB, MEDIUMBLOB, LONGBLOB  -->  []byte
                                 BIT  -->  []byte
                           SET, ENUM  -->  []byte
                                NULL  -->  nil
```
