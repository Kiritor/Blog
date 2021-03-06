layout: post
title: Golang:go get现实显示进度(转)
category: golang
tags: [go get,golang]
---
&nbsp;&nbsp;&nbsp;&nbsp;golang在使用go get下载package时,有些package如果比较大的话,下载时间比较的久(会给我们造成卡死的情况),希望可以有一个下载进度。google一下,已经有人解决了,自己也尝试了下,发现没问题,果断转载过来了.

&nbsp;&nbsp;&nbsp;&nbsp;原文地址如下:[http://www.leanote.com/blog/view/544ca9a824ab0a1dbd000000](http://www.leanote.com/blog/view/544ca9a824ab0a1dbd000000)
<!--more-->
&nbsp;&nbsp;&nbsp;&nbsp;因为leanote在github.com上的包有点大, 所以 go get github.com/leanote/leanote/app 会很慢, 这个会执行几分钟或更长, 不知道的朋友还以为卡死了. 找了下 go get 没有一个选项可以输出进度的, 于是决定修改golang源码(别以为很有技术含量, 还不是go代码?).

&nbsp;&nbsp;&nbsp;&nbsp;看了下golang的源码 src/cmd/go 下是go命令的源码, 其中, get.go是go get命令的代码, build.go 是go build的代码.

&nbsp;&nbsp;&nbsp;&nbsp;刚开始走了点弯路, 想着改变get.go来显示进度, 无果之后想了下, go get 其实就是调用git , hg, svn的命令从仓库中下载的, 由此思路找到vcs.go(vcs全称为version control system), 果然这里面包含了调用git, hg, svn的命令. 问题迎刃而解:

```bash
	修改git clone命令, 添加 --progress选项, 使其输出进度
	修改cmd.Run()执行的地方, 使其将输出定位到标准输出流上
```

## 1. 修改git clone命令
&nbsp;&nbsp;&nbsp;&nbsp;找到如下代码, 在createdCmd修改为 clone --progress {repo} {dir}.其它命令hg, svn...添加进度方法类似.
```go
	// vcsGit describes how to use Git.
	var vcsGit = &vcsCmd{
	name: "Git",
	cmd:  "git",

	createCmd:   "clone {repo} {dir}", // 此处修改为 clone --progress {repo} {dir}
	downloadCmd: "pull --ff-only"
}
```
## 2. 重定向输出流
&nbsp;&nbsp;&nbsp;&nbsp;找到run1()方法, 在 cmd.Stderr = &buf 下添加两行, 如:
```go
	var buf bytes.Buffer
	cmd.Stdout = &buf
	cmd.Stderr = &buf
	cmd.Stdout = os.Stdout // 重定向标准输出
	cmd.Stderr = os.Stderr // 重定向标准输出
	err = cmd.Run()
```
&nbsp;&nbsp;&nbsp;&nbsp;Ok, 搞定, 接下来执行golang源码 src下的 all.bash 重新编译golang, 编译要些时间, 编译完后使用go get 试试:

[http://leanote.com/file/outputImage?fileId=544caf469bf7f1285400005a](http://leanote.com/file/outputImage?fileId=544caf469bf7f1285400005a)

&nbsp;&nbsp;&nbsp;&nbsp;看到进度条就不用担心了吧.

&nbsp;&nbsp;&nbsp;&nbsp;之前修改golang源码使其关闭变量未使用, 包未使用的错误 : 

[关闭golang的 variable declared but not used 和 package imported but not used](http://leanote.com/blog/view/53118d331a9108428c000001)

## Tips
&nbsp;&nbsp;&nbsp;&nbsp;linxu、mac下执行all.bash重新编译golang,windows下执行all.bat貌似有问题,可以执行run.bat即可.下面是我使用go get的情况:
```bash
	Cloning into '/Users/lcore/dev/code/go/src/github.com/kiritor/ini'...
	remote: Counting objects: 36, done.
	remote: Total 36 (delta 0), reused 0 (delta 0), pack-reused 35
	Unpacking objects: 100% (36/36), done.
	Checking connectivity... done
	b577c9a266676ad31bd94824e2b1a80052f4f33a refs/heads/master
	b577c9a266676ad31bd94824e2b1a80052f4f33a refs/remotes/origin/HEAD
	b577c9a266676ad31bd94824e2b1a80052f4f33a refs/remotes/origin/master
	Already on 'master'
```