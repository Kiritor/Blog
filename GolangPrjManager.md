title: "Golang工程管理"
date: 2015-04-09 09:34:07
tags: [Go,工程管理]
categories: golang
---
#Go工程管理
&#160; &#160; &#160; &#160;接触Go也有一段时间了,也写了一些零星的代码,零星的代码无法像工程一样管理起来让我有一种挫败感.而且实际的开发中,也不可能只有一个单一的源文件,逐步编译无意于一场灾难.因此有必要学习一下Go是如何来管理工程的了.
&#160; &#160; &#160; &#160;Go工程管理的一个亮点在于消除了工程文件的感念,完全用目录结构和包名来推导工程结构和构建顺序.Go自身提供了良好的工程化管理,几乎不依赖于IDE.

#相关概念
&#160; &#160; &#160; &#160;在详细实践GO的工程管理之前,先理解下相关概念,这有助于我们理解Go的工程管理:<span class="color">安装路径</span>、<span class="color">官方包路径</span>、<span class="color">项目路径</span>
&#160; &#160; &#160; &#160;Go中只有两个路径:
```go
    GOROOT:go安装路径,官方包路径根据这个设置自动匹配
	GOPATH:工作路径
```
&#160; &#160; &#160; &#160;<span class="color">Tips:</span>Go中是没有项目路径这个概念的,只有包.可执行包只是特殊的一种,即是我们常说的项目,GOPATH可以设置多个(不建议).不管是可执行包还是非可执行包,都应该置于$GOPATH/src下.
&#160; &#160; &#160; &#160;对于$GOBIN,在之后实践的时候再细说.
#Go项目目录
&#160; &#160; &#160; &#160;Go官网中有过介绍,Go项目的目录结构一般包含以下几个:
* **src:包含项目的源文件**
* **pkg:包含编译后生成的包/库文件**
* **bin:包含编译后生成的可执行文件**
&#160; &#160; &#160; &#160;下面,一步步来看Go是如何进行工程目录的组织管理的,不过在此之前,我们要先配置好环境变量,以及工程目录结构,这里我的工程目录结构是:(GOPATH=E:\code\GoProject)
<code>
     bin
	   ...
	 pkg
	   ...
	 src
	   ...
</code>
#第一个执行程序(包)、项目
&#160; &#160; &#160; &#160;前面已经说了,Go的可执行package就项目与项目.建一个可执行包hello(项目),其中包含一个源文件hello.go,目录结构如下:
<code>
     bin
	    ...
	 pkg
	    ...
     src
	    hello
		  |-hello.go
</code>
&#160; &#160; &#160; &#160;代码如下:
```go
		package main

		import (
			"fmt"
		)

		func main() {
			fmt.Println("hello")
		}
```
##go build
&#160; &#160; &#160; &#160;使用go build,你会发现并没有按照期待的,生成在bin目录下:
```go
     go build src/hello                        生成在根目录下
	 cd src/hello  go build hello.go           生成在子包目录下
```
&#160; &#160; &#160; &#160;正确的做法是使用go install命令:
```go
     go install hello
```
&#160; &#160; &#160; &#160;之后的目录结构:
<code>
     bin
	    |-hello.go      //bin中生成执行程序
	 pkg
	    ...
     src
	    hello
		  |-hello.go
</code>
&#160; &#160; &#160; &#160;<span class="color">Tips:</span>需要注意的是如果设置了环境变量GOBIN,hello程序将被安装到GOBIN目录
#第一个程序库(library)
&#160; &#160; &#160; &#160;上面讲了一个可执行包(项目)的管理,接下来看看一个非可执行包(lib库)的管理.首先创建一个库utils,目录结构如下:
<code>
     bin
	    |-hello.exe      //bin中生成执行程序
	 pkg
	    ...
     src
	    hello
		  |-hello.go
		utils
		  |-utils.go
</code>
&#160; &#160; &#160; &#160;代码如下:
```go
		package utils

		import (
			"fmt"
		)

		func System(str string) {
			fmt.Println(str)
		}
```
&#160; &#160; &#160; &#160;同样的,我们使用go install命令进行编译,之后的目录结构如下:
<code>
     bin
	    |-hello.exe      //bin中生成执行程序
	 pkg
	    |-windows-amd64
		  |-utils.a
     src
	    hello
		  |-hello.go
		utils
		  |-utils.go
</code>
&#160; &#160; &#160; &#160;在其它包使用它也很简单,一个例子:
```go
		package main

		import (
			"utils"
		)

		func main(){
			utils.System("hello")
		}
```
&#160; &#160; &#160; &#160;上面提及了go build、go install,两者有什么区别呢?
* **go build:编译包,如果是main包则在当前目录下生成可执行文件,其他包不产生.a文件;**
* **go install:编译包,同时复制结果到$GOPATH/bin,$GOPATH.pkg等对应的目录下;**
* **go run gofiles编译列出的文件,并生成可执行文件然后执行,只能作用于main包,否则会出现go run:cannot run non-main pakcage的错误.**
&#160; &#160; &#160; &#160;此外应该知道的是,go run是不需要设置$GOPATH的,但go build、go install必须设置,所以go run常用来测试一些功能,这些代码一般不包含在最终的项目中(生产环境中)
#自动化文档
&#160; &#160; &#160; &#160;Go的注释可以自动转换成文档.只需要将注释写在package、func之前,中间不能有空格,简单的例子:
```go
	//utils工具包
	package utils

	import (
		"fmt"
	)

	//输出字符串到控制台
	func System(str string) {
		fmt.Println(str)
	}
```
&#160; &#160; &#160; &#160;在代码中这样写入注释之后,就可以使用godoc utils查看文档了:
```bash
		PACKAGE DOCUMENTATION

		package utils
			import "utils"

			utils工具包

		FUNCTIONS

		func System(str string)
			输出字符串到控制台
```
#测试框架
&#160; &#160; &#160; &#160;Go本身是支持测试的,测试源码的命名必须采用如下格式,建议以文件未单位,对应的测试文件未fileName_test.go
&#160; &#160; &#160; &#160;测试源码必须包含testing,测试函数必须按照"TestFuncName"的方式命名,一个简单的测试例子.
```go
		package utils

		import (
			"testing"
		)

		func TestSystem(t *testing.T) {
			//
			System("test")
		}
```
&#160; &#160; &#160; &#160;之后运行如下命令进行测试:
<code>
go test uitls   //测试指定包的测试程序
go test         //执行所有测试程序
</code>
&#160; &#160; &#160; &#160;测试结果像这样:
```bash
		E:\code\GoProject>go test utils
		ok      utils   0.111s
```
#Benchmark
&#160; &#160; &#160; &#160;Benchmark是测试框架的一部分,用于函数的运行效率相关信息.
&#160; &#160; &#160; &#160;以刚才的例子为例,首先在uitls中添加如下函数:
```go
		func Add(var1,var2 int) int {
			return var1+var2
		}
```
&#160; &#160; &#160; &#160;之后在测试文件中添加Benchmark测试代码:
```go
		func BenchmarkMulti(b *testing.B) {
			for i:=0;i<b.N;i++ {
			   Add(1,2)
			}
		}
```
&#160; &#160; &#160; &#160;执行test,并执行Benchmark:go test utils -bench=BenchmarkMulti ,执行结果如下:
```go
		E:\code\GoProject>go test utils -bench=BenchmarkMulti
		test
		PASS
		BenchmarkMulti  2000000000               1.56 ns/op
		ok      utils   3.390s
```
&#160; &#160; &#160; &#160;Tips:如果测试没有通过,Benchmark也不会执行,不过我们可以通过如下命令跳过test,直接执行Benchmark:go test uitls -run=XXX -bench=BenchmarkMulti,执行结果如下:
```GO
		E:\code\GoProject>go test utils -run=XXX -bench=BenchmarkMulti
		PASS            //test直接被pass掉了
		BenchmarkMulti  2000000000               1.85 ns/op
		ok      utils   3.986s
```
#最佳实践
1.  做好GO工程的管理(包的管理),不然杂乱无章的代码,会有深深的挫败感.
2.  建议使用一个$GOPATH,一个项目一个$GOPATH并不可取,包无法很好的管理起来.
3.  尽量使用go install,而不是go build,这样能够规范项目的整体结构(不会造成编译生成的可执行文件到处乱跑)go install作用于包级别
#注意事项
&#160; &#160; &#160; &#160;Go工程管理的时候,稍有不慎就会出现错误,下面是一些比较常见的错误.
##一个文件夹下面包含多个不同包的源文件
&#160; &#160; &#160; &#160;这种情况下执行go install或者go build ,go run 都会错误:
```bash
		can't load package: package myfunc: found packages myfunc (a.go) and main (b.go)
```
&#160; &#160; &#160; &#160;<span class="color"><B>每个子目录只能存在一个pakcage,否则会报错.但是可以存在多个源文件,(属于同一package)</B></span>
##一个项目不能包含多个main()函数
&#160; &#160; &#160; &#160;实际上编译是没有问题的,但是这并不是一种正确的做法,一个项目只能有一个入口函数.
#参考资料
&#160; &#160; &#160; &#160;[how-to-write-benchmarks-in-go](http://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go)
&#160; &#160; &#160; &#160;[Running multi-file main package](https://groups.google.com/forum/?fromgroups=#!searchin/golang-nuts/why$20use$20go$20install/golang-nuts/qysy2bM_o1I/khIRe3AJ_wEJ)