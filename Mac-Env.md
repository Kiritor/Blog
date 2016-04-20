layout: post
title: Mac设置环境变量
category: work
date: 2016-04-18 23:34:07
tags: [mac]
---
在mac,linux中配置环境变量对于新手来说,是一个有点头痛的问题.因为经常看到不同的方式配置环境变量.到底应该怎么配置,配置在什么地方(全局/用户级).做个简要的笔记.
## Shell类型
首先需要判断下使用的Mac OS X是什么样的Shell,使用命令echo $SHELL
如果输出的是:csh或者tcsh.那么就是C Shell.
如果输出的事:bash,sh,zsh,那么就是Bourne Shell的一个变种.
Max OS X 10.2之前默认的是C Shell.
Mac os X 10.3之后默认的是Bourne Shell.
## Mac环境变量配置
如果是Bourne Shell设置环境变量的地方如下
### ./etc/profile
全局(共有配置),不管是哪个用户,登陆时都会读取该文件。不建议修改此文件.
### ./etc/bashrc
全局(共有)配置,bash shell执行时，不管何种方式,都会读取次文件。一般在这个文件中添加系统级别的环境变量.
### ~/.bash_profile
每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!一般在这个文件添加用户级的环境变量,比较建议使用该方式配置。
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_25.jdk/contents/Home
MAVEN_HOME=/Users/lcore/dev/application/apache-maven-3.2.5
PATH=$PATH:$MAVEN_HOME/bin

export MAVEN_HOME
export PATH
export GOPATH=~/dev/code/go/
#export GOBIN=/usr/local/go/bin
export PATH=$PATH:$GOBIN
```
如果想立即生效,即可执行下面的语句:
```bash
source .bash_profile
```
