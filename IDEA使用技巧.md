title: IDEA使用技巧
layout: post-noTOC
date: 2016-04-02 10:20：34
tags: IDEA
categories: 技巧
---
## 写在前面
工作中一直使用Eclipse作为开发工具,自己下来玩的时候用IDEA居多,然而有时候一些快捷键啊,配置操作啊,容易忘.当时在搜索又比较耗时间,而且网上的文章也不够直接,好记性不如烂笔头,自己做一个记录列表.
## 自动import
IDEA没有类似Eclipse的自动导入清除无用包快捷键optimize imports(ctrl+shift+o),但是可以通过设置让IDE支持自动导入与删除已经明确的包.如下图所示:
<!--more-->
![自动导入](http://o7q5y55yj.bkt.clouddn.com/idea%20import.png)
Tips:需要注意的是如果存在不明确的包,需要手动确认.
## maven skip test
maven编译、打包的时候如果编写了单元测试,会先运行单元测试,通过了在继续编译、打包,一般会选择跳过编译.在maven命令面板中点击如下图按钮即可跳过编译.
![跳过测试](http://o7q5y55yj.bkt.clouddn.com/mavenskiptest.png)