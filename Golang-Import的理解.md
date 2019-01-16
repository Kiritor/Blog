title: Golang Import的理解
date: 2015-04-03 14:59:14
tags: [golang,import]
categories: golang
---
初学go不久,在使用beego的时候发现了不同的import方式,于是查阅相关资料,做个备注。
初期我们编写go语言的时候经常使用import这个命令来导入包文件,导入的方式一般如下方式:
```bash
import (
	"fmt"
)
```
<!--more-->
之后通过如下的方式调用
```bash
fmt.Println("go")
```
我们知道fmt是Go语言的标准库,他其实是去goroot下加载该模块。而且Go语言提供两种加载模块的方式:<strong>相对路径</strong>、</strong>绝对路径</strong>
```bash
import "./model"    --相对路径,不推荐
import "XXX/model"  --个gopath/src/XXX下的model模块,最常见
```
不过,在阅读某些开源项目时,会看到一些令人费解的import方式,下面来一一探寻下:
# 点操作
我们有时候可以看到如下的导入方式
```bash
import (. "fmt")
......
Println("go")
```
这个点操作的含义就是这个包导入之后,使用该包函数的使用可以省略前缀
# 别名操作
别名操作可以把包名换成一个容易记忆的名字(对于那些包名过长的)
```bash
import (f "fmt")
...
f.Println("go")
```
# "_"操作
这个操作比较令人费解,beego里面就有使用到(bee new xx生成的)
```bash
package main

import (
	_ "beego_cms/routers"
	"github.com/astaxie/beego"
)

func main() {
	beego.Run()
}
```
"_"操作其实是引入该包,但是并不直接使用包里面的函数,而是调用该包里面的init函数,如何理解这个问题?通过如下的图,来理解一下go包的加载机制:
<center>![go_import](http://kiritor.github.io/img/go_import.png)</center>
程序的初始化和执行都起始于main包。如果main包还导入了其他包,那么就会在编译时将他们一次导入。同一个包只会被导入一次。当一个包被导入时,如果还包含了其他包,那么会先将其他包导入进来,然后在对这些包中的包及常量和变量进行初始化,接着执行init函数(如果有的话),依次类推。等所有的包都导入完了,就开始对main包中的包级常量和变量进行初始化,然后执行main包里面的init函数(若果有),最后执行main函数。

_操作只是将该包引入了,只初始化里面的init函数和一些变量,但是往往这些init函数里面是注册自己包里面的引擎,让外部可以方便的使用。和很多实现database/sql的一样,在init函数里面都是调用了sql.Register(name string,driver driver.Driver)注册自己,然后外部可以使用了。

原文地址:[http://blog.beego.me/blog/2013/07/27/golang-import-shi-yong-ru-men/](http://blog.beego.me/blog/2013/07/27/golang-import-shi-yong-ru-men/)