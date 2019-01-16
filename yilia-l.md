---
title: hexo-theme-yilia-l
date: 2016-04-28 14:55:42
tags: [hexo]
categories: front
---
## 介绍
github上通过hexo搭建了个人博客,闲时写写文章. 没事改改博客的样式,修改自[yilia](https://github.com/litten/hexo-theme-yilia)主题.个人十分喜欢yilia主题的简洁、优雅的两栏风格，极致的性能体验. 不过其界面的排版不是非常满意,因此就自己改装了下,调整了相关样式,新增了一些新的功能.
>    1.状态栏(最近的一些状态)
>    2.微简历(简单的介绍信息,只在首页显示,可通过配置文件关闭)
>    3.文章目录(目录通过按钮动态显示,且可以通过指定layout类型为post-noTOC不显示目录)
>    4.多说最新评论(首页悬浮显示,可通过配置文件关闭)

Github地址:[https://github.com/Kiritor/hexo-theme-yilia-l](https://github.com/Kiritor/hexo-theme-yilia-l)
<!--more-->
## 使用
### 安装
使用如下命令安装:
```bash
$ git clone https://github.com/Kiritor/hexo-theme-yilia-l.git themes/yilia-l
```

### 启用主题
修改hexo博客根目录下的_config.yml:
```bash
theme:yilia-l
```
### 更新主题
```bash
cd themes/yilia-l
git pull
```
## 界面外观
相关的界面截图如下:
### 宽屏
<center>![首页](/img/yilia-l-index-k.png)</center></br>
<hr>
<center>![文章页](/img/yilia-l-post-k.png)</center>
### 窄屏
<center>![首页](/img/yilia-l-index-z.png)</center></br>
<hr>
<center>![文章页](/img/yilia-l-post-z.png)</center></br>
### 移动端
<center>![移动端](/img/yilia-l-mobile.png)</center>
## 主題配置
主题的配置信息如下,相关的配置信息见注释,至于其他的配置信息和大多数hexo主题一样,并无太大区别.
```bash
# Header
#定义菜单
menu:
  #主页,所有文章
  主页: /
  前端开发: /categories/front
  工作日志: /categories/work
  点滴生活: /categories/life

#定义子菜单
subnav:
  github: "https://github.com/Kiritor"
  weibo: "http://weibo.com/3206206100/profile?topnav=1&wvr=6"
  rss: "#"
  zhihu: "http://www.zhihu.com/people/kiritor"
  #douban: "#"
  #mail: "#"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

rss: /atom.xml

# Content
excerpt_link: More
archive_yearly: true #按年存档
fancybox: true
mathjax: true

# Miscellaneous
google_analytics: ''
favicon: /img/leaves.ico

#你的头像url
avatar: "http://kiritor.github.io/img/lcore.jpg"
#是否开启分享
baidushare: true
#是否开启云标签
tagcloud: true

#是否开启多说
duoshuo_shortname: kiritor
#是否开启多说最近评论:悬浮于首页
recentComment: true

#是否开启最近通知
recent: true
recentContent: 最近研究shiro中......

#是否开启微简历
resume: true

#是否开启友情链接
#不开启——
friends: false
#开启——
#friends:
  #奥巴马的博客: http://localhost:4000/
  #卡卡的美丽传说: http://localhost:4000/
   #本泽马的博客: http://localhost:4000/
  #吉格斯的博客: http://localhost:4000/
  #习大大大不同: http://localhost:4000/
  #托蒂的博客: http://localhost:4000/

#是否开启“关于我”。
#不开启——
#aboutme: false
#开启——
aboutme: 乐于分享,喜欢环境音乐,爱看动漫,偶尔写写技术博客的小程一枚...

```
