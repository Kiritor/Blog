layout: post
title: Mac实用小技巧
category: work
date: 2016-04-18 23:34:07
tags: [mac]
---
使用Mac已经很长一段时间了,从最开始的生疏到慢慢熟悉,掌握了一些实用的小技巧,这里略作整理,不定期更新.
### 终端忽略大小写补全
打开终端,输入:
<!--more-->
```bash
nano .inputrc
```
在里面粘贴如下语句:
```bash
set completion-ignore-case on
set show-all-if-ambiguous on
TAB: menu-complete
```
Control+O,回车保存,重启中断,OK~,之后按Tab键即可自动补全~
