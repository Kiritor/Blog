title: Windows下搭建go语言开发环境
date: 2015-03-27 15:52:33
tags: [go]
categories: golang
author: LCore
---
# 前言
博客打算从SAE转移到github上,须花些时间。这段时间一直在学习go语言,还在入门阶段,不过也算了解了一些语法。由于最近博客搬家,也没啥可写的,就写一下自己开发go语言的配置吧。
**Bracket + Git**
由于初学并没有采用集成开发环境(liteIDE,eclipse等),因为我想熟悉下go的相关命令,以及go工程项目的管理。
<!--more-->
# GO语言
Go语言是Google内部主推的语言,它作为一门全新的静态类型开发语言,与当前的开发语言相比具有许多令人兴奋不已的新特性。专门针对多处理器系统的应用编程进行了优化,使用go语言完全可以媲美c、c++的速度,而且更加安全、简洁,支持并行进程。
以下是go语言的主要特征:
     1. 自动垃圾回收
	 2. 更丰富的内置类型
	 3. 函数多返回值
	 4. 错误处理
	 5. 匿名函数和闭包
	 6. 类型和接口
	 7. 并发编程
	 8. 反射
	 9. 语言交互性
# Window开发环境搭建
以上基本都属于废话,搭建开发环境尽快开启编程之旅才是王道,笔者采用Brackets + Git的方式进行学习。使用Brackets主要在于自己一直用的是Brackets,且初学建议先别用IDE,不过sublimeText也是一个非常好的选择。
## 1、安装Golang的SDK
Google的官网被墙,下载地址如下:[http://www.golangtc.com/download](http://www.golangtc.com/download),请下载对应系统的版本,之后傻瓜式的下一步安装即可。
安装完成之后,打开终端,输入go、或者go version出现如下信息即表示安装成功:
```bash
   go version go1.4.1 windows/amd64
```
## 2、配置环境变量
安装完SDK之后接下来便是配置环境变量了,windows下配置信息如下:
```bash
   GOROOT=D:\go\     --go根目录
   GOPATH=E:XXXXX    --日常开发的根目录,可以配置多个";"分隔
```
重新打开cmd,输入go env可以查看配置后的效果:
```bash
   set GOARCH=amd64
   set GOBIN=
   set GOCHAR=6
   set GOEXE=.exe
   set GOHOSTARCH=amd64
   set GOHOSTOS=windows
   set GOOS=windows
   set GOPATH=E:\code\GoStudy\music;E:\code\GoStudy\calcproj;
   set GORACE=
   set GOROOT=D:\Go
   set GOTOOLDIR=D:\Go\pkg\tool\windows_amd64
   set CC=gcc
   set GOGCCFLAGS=-m64 -mthreads -fmessage-length=0
   set CXX=g++
   set CGO_ENABLED=1
```
## 开发工具配置(Brackets)
Brackets是Adobe公司的一款前端开发工具,和sublime非常相似(但是就功能上来说弱于sublime),不过笔者更喜欢Brackets的风格,也玩了很长一段时间,因此就决定是它了。
官网下载地址:[Brackets.io](https://github.com/adobe/brackets/releases),选择合适的版本下载安装即可。
Brackets本质上是一个文本编辑器,但是可以通过配置插件的方式打造成一个媲美IDE的工具,下面是我使用到的一些插件,安装插件的方式也很简答,右侧有一个插件管理器,打开即可进行安装:
```bash
1. brackets-code-folding            代码折叠插件
2. brackets-css-shapes-editor       css插件
3. brackets-jsbeautifier            代码美化插件
4. drewkoch.icons                   文件类型图标插件
5. go-ide                           golang插件(关键字高亮)
6. gruehle.brackets-autoprefixer    自动补全插件(golang不行)
7. themesforbrackets                主题插件
8. websiteduck-wdminmap             右侧minmap插件(类似sublime)
9. zaggino.brackets-git             git插件
```
其他插件的配置都不复杂这里就不一一介绍了,重点介绍一下git插件的配置。当安装好git插件之后,右侧会出现如下图标(为了便于排版,我将图片旋转):

<center>![git](http://kiritor.github.io/img/git.png)</center>

红线标出的第一个就是git插件了(可以调出git界面),第二个是打开控制台的快捷按钮,等会介绍如何配置git插件使其显示出来。我们点击git图标即可在界面下方唤出git界面如下:
<center>![git](http://kiritor.github.io/img/git_mark.png)</center>

之后点击设置图标,进行相关设置:

<center>![git](http://kiritor.github.io/img/git1.png)</center>
对于插件的使用我也就不多提了,非常简单,熟悉下即可上手了

## 附图
最后秀一下最后的效果:
<center>![git](http://kiritor.github.io/img/brackets.png)</center>

# GO小试
配置好开发环境之后,简单的进行一下go语言的开发。新建一个文件夹,并用brackets打开,新建main.go文件即可进行编辑了。
```bash
package main

import (
	"fmt"
)

func main() {
	fmt.Println("hello go");
}
```
代码编写完后,打开控制台,定位到当前目录下,输入go build main.go即可完成编译,之后直接main(window下即可查看运行结果),也可以直接运行go run main.go直接运行(不产生exe文件)

到目前为止已经安装好了开发golang的程序的基本环境,可以开心的享受golang的奇妙之处了!


