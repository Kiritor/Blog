title: JQuery EasyUi框架学习
date: 2015-03-26 22:45:08
tags: [jquery,easyui]
categories: front
---
#前言
新项目的开发前端技术打算采用EasyUI框架,项目组长将前端EasyUI这块的任务分配给了我.在进行开发之前,需要我这菜鸟对EasyUI框架进行一些基础的入门学习。之后会在学习的过程中将自己遇到的问题和有用的东西记录下来。
#关于EasyUI
EasyUI框架是基于JQuery的,使用它帮助我们快捷的构建web网页。EasyUI框架是一个简单、易用、强大的轻量级web前端javascript框架。现阶段来说,在开发web项目时,前端的开发我们更喜欢使用jQuery代替原生的javascript,原因大概如下方面:
<!--more-->
      1. JQuery是基于js的扩展,使页面和脚本进行了分离。
	  2. JQuery的宗旨:用最少的代码做更多的事。
	  3. 在javascript的库中,JQuery对性能的理解是十分到位的。
	  4. 基于JQuery的插件越来越多,也越来越人性化,对于项目的开发有一定的借鉴。
	  5、简单、易学、快速上手让其更具吸引力。
#EasyUI的优点
EasyUI是基于JQuery的框架,相比于其他框架来说其具有如下的优点:
      1. EasyUI可以很容易的帮助创建web页面。
	  2. EasyUI是基于用户界面的插件集合。
	  3. EasyUI提供了构建更加现代化,交互性的js程序应用。
	  4. 通过EasyUI我们可以编写一些html标记来定义用户界面。
	  5. 极大的提升项目开发的时间。
EasyUI的使用
最基本的使用就是引入easyui.min.js文件，以及其对应的jquery.min.js文件。注意的是jquery-easyui是基于jquery的,因此jquery.min.js文件最好先于easyui.min.js先引入。 
``` bash
	  <script type="text/javascript" src="jquery-easyui-1.3.6/jquery.min.js"></script>
	  <script type="text/javascript" src="jquery-easyui-1.3.6/jquery.easyui.min.js"></script>
```
前面也说道了,EasyUI是插件集合,需要用到的时候在自行引入就可以了。
#EasyUI的下载和学习
EasyUI的下载推荐EasyUI官网:http://www.jeasyui.com/ EasyUI的学习,官网上已经提供了非常丰富的学习实例,而且Demo非常丰富。这里推荐几个有用学easyui的站点：
      1. EasyUI官网
      2. JQueryEasyUI中文社区:http://bbs.jeasyuicn.com/(张宇、夏季)
      3. EasyUI学习班:http://www.jeasyuicn.com/
      4. EasyUI入门视频教程(张宇前辈)
	  5. ......
好了,前期的准备已经差不多了,后续就是一点一点深入学习了。