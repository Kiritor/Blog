title: EasyUI基础入门之easyloader(加载器)
date: 2015-03-27 10:39:13
tags: [easyui,loader]
categories: front
---
在了解完easyui的parser(解析器)之后,接下来就是easyloader(简单加载器)的学习了。
# 什么是EasyLoader
正如其名字一样easyloader的作用是为了动态的加载组件所需的js文件,这体现了EasyUI作为轻量级框架对性能的合理掌握（可以动态的加载所需组件）,不过一般而言很少使用到easyloader(会给使用者带来一定的难度)。那么使用EasyLoader的场景有哪些呢？
<!--more-->
# EasyLoader的使用场景
    ● 出于性能的考虑,不一次性的加载easyui核心js、css文件,而是先展示基础文档结构。
    ● 项目只是简单的用到easyui的几个组件,此时可以按需加载该组件的js和css文件。
    ● 你需要使用某个组件,但是不知道该组件是否依赖于其他组件(简单的js引用无法达到),使用easyloader可以自动加载依赖组件。
    ● 你需要把JQuery基础库和自己实现的js结合起来，以达到更好的展示性能。 
# EasyLoader加载器
简单的了解了什么是easyloader以及其大概的使用场景,接下来看看easyloader的属性、事件和方法。
# properties
easyloader有7个属性，具体可以参加官网
# Event
加载器包含两个事件,具体参见官网
# Method
加载器只有一个方法:load,其参数为module,callback(回调函数)，载入特定的模块，当载入的成功的时候调用该回调函数有效的模块参数可以使一个单一的模块名称、存储模块名称的数组、css样式文件、js脚本文件。
# EasyLoader使用
接下来我们着眼于easyloader如何使用，通过上面属性表中的modules，不难猜到这个属性就是easyui定义模块用的。modules本质上来说是一个json格式对象。其形式如下: 
```bash
      modules = {
      draggable:{
         js : "jquery.draggable.js",
         css : "pagination.css",
         dependencies : ["linkbutton"]
       }
    };
```
key值即是定义的模块名称,值又是一个json对象,包含三个属性js、css、dependencies。js就是模块需要导入的js名称,css是该模块的样式,dependencies定义该模块的依赖模块。
上面定义了一个模块,接下来谈谈该如何添加我们自定义的模块,并且通过easyloader进行加载。

第一步:我们先要打开easyloader.js文件,阅读代码(压缩过)我们可以简单的看出来easyui加载的时候到底加载了哪些模块,“_1”是easyui已经定义好的模块.以及各个模块之间的依赖关系。这样,我们通过修改easyloader.js的代码就可以有选择性的加载所需的模块,提高easyui的性能(一般情况下不建议)。那么我们如何简单的定义下自己的模块呢? 我们需要改造一下easyload.js，我们将easyloader.js代码中的所有_1变量替换为easyloader.modules，不过最后一个"modules:_1"的引用不要修改。

第二步,在easyui原有的模块基础上,我们扩展它,加入自己的模块。
```bash
		easyloader.modules = $.extend({},{
			"test":{
				js:'test.js'		css:'test.css'	}
		},easyloader.modules);
```
上面的代码在原有easyloader.modules的基础上在增加了一个test模块并且定义了模块的js和css。这样我们增加的第一个自定义模块就添加完成了。使用的方式和easyloader加载其他模块一样。

Tips:我们自己定义的js和css文件必须是绝对路径。
```bash
	easyloader.load('mymodule', function(){      
	 //do something
	});
```
