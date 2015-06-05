title: EasyUI基础入门之Draggable(拖拽)
date: 2015-03-27 11:35:01
tags: [easyui,draggable]
categories: front
---
前面学习了easyui基础的解析器,加载器。对于他们入门阶段我们只需简单的了解下即可，毕竟先阶段并不会太过深入。接下来根据easyui官网文档的顺序安排学习下Draggable插件。
#Draggable是什么
Draggable是easyui中用于实现拖拽功能的一个插件。利用它，我们可以实现控件的拖拽效果。
Draggble覆盖默认值$.fn.draggable.defaults。 
<!--more-->
#Draggable
下面看看官网对于Draggable的描述吧。 
##属性
其属性请参阅官网
一个简单的例子,代码如下:
```bash
    <!DOCTYPE html>
	<html>

	<head>
		<meta charset="UTF-8">
		<title>Basic Draggable - jQuery EasyUI Demo</title>
		<link rel="stylesheet" type="text/css" href="jquery-easyui-1.3.6/themes/metro/easyui.css">
		<link rel="stylesheet" type="text/css" href="jquery-easyui-1.3.6/themes/icon.css">
		<link rel="stylesheet" type="text/css" href="jquery-easyui-1.3.6/demo/demo.css">
		<script type="text/javascript" src="jquery-easyui-1.3.6/jquery.min.js"></script>
		<script type="text/javascript" src="jquery-easyui-1.3.6/jquery.easyui.min.js"></script>
	</head>

	<body>
		<h2>Basic Draggable</h2>
		<p>Move the boxes below by clicking on it with mouse.</p>
		<div id="dd" class="easyui-draggable" data-options="handle:'#title'" style="width:100px;height:100px;">
			<div id="title" style="background:#ccc;width:100px;height:100px;">容器里面的内容</div>
		</div>
		<script>
			$(function () {

				$("#title").draggable({
					proxy: function (a) {
						var a = $('<div class="proxy_div">你拖我干啥</div>');
						a.appendTo('body');
						return a;
					},
					cursor: 'pointer',
					edge: '6'
				});
			});
		</script>
	</body>

	</html>
```
##事件
 Draggable的事件还是比较好理解的，具体参考官网:
 ##方法
 参阅官网
 ##使用方式
 两种使用的方式：
 1、通过html标记创建,方法如下:
 ```bash
	 <div id="dd" class="easyui-draggable" data-options="handle:'#title'" style="width:100px;height:100px;">
			<div id="title" style="background:#ccc;width:100px;height:100px;">容器里面的内容</div>
	 </div>
 ```
 2、通过js脚本创建:
 ```bash
    <div id="dd" style="width:100px;height:100px;">
    <div id="title" style="background:#ccc;">title</div>
	</div>
	<script>
		$('#dd').draggable({
			handle: '#title'
		});
	</script>
 ```
#Demo
对于Draggable，官网提供了一些个例子地址如下:http://www.jeasyui.com/demo/main/index.phpplugin=Draggable&theme=default&dir=ltr&pitem=
初学来说的话,上述的demo例子就够了。over!