title: EasyUI基础入门之Parser(解析器)
date: 2015-03-26 23:58:08
tags: [EasyUI,Parser]
categories: front
---
#前言
JQuery EasyUI提供的组件包括功能强大的DataGrid,TreeGrid、面板、下拉组合等。用户可以组合使用这些组件,也可以单独使用其中一个。(使用的形式是以插件的方式提供的)
<!--more-->
#EasyUI体系结构
EasyUI所有的插件主要分为六大部分。Base基础、Layout布局、Menu&Button、Form表单、Window窗口等。从最基础的开始先掌握EasyUI基础部分。Base部分包含了八个基础插件分别为:
    1. parser(解析器)
    2. easyloader(加载器)
    3. draggable(拖动)
    4. droppable(放置)
    5. resizable(大小调整)
    6. pagination(分页)
    7. progressbar(进度条)
    8. searchbox(搜索框) 
我们先从parser开始。
#Parser(解析器)
解析器是easyui非常重要的基础组件,在easyui中我们能够简单的通过class定义一个组件,从而渲染出非常好的交互效果。就是通过parser进行解析的。parser会获取所有在指定范围内定义为easyui组件的class定义,并且根据后缀定义把当前节点解析渲染成特定的组件。
parser可以有两种使用方法: 
```bash
   1、$.parser.parse();不带参数,默认解析该页面中所有被定义为easyui组件的节点。 
   2、$.parser.parse('#cc');带一个jquery选择器，使用这种方式我们可以单独解析局部的easyui组件节点。
```
不过这里要说明的是这个jquery选择器必须是你解析组件的父级以上的节点。也就是说这个查找出来的节点相当于一个容器,它只会解析容器里面的内容。
parser属性:
      
名称	             类型	     描述	                   默认值
$.parser.auto 	boolean	定义是否自动解析easyui组件	true

名称	参数	描述
$.parser.onComplete 	context	当语法解析完成之后触发的event

```bash
<html>
<head>
    <title>EasyUI基础之Parser</title>
    <script type="text/javascript" src="js/jquery.min.js"></script>
    <script type="text/javascript" src="js/jquery.easyui.min.js"></script>
    <script>
        function closes() {
            $("#Loading").fadeOut("normal", function () {

                $(this).remove();
                alert("数据加载完成");

            });
        }
        var pc;
        $.parser.onComplete = function () {
            if (pc) {
                clearTimeout(pc);
            }
            pc = setTimeout(closes, 1000);

        }
    </script>
</head>
<body>
<div id='Loading'>
    <image src='images/loading.gif'/>
    <font color="#2bd4cd" size="4">页面正在加载中···</font>
</div>
</body>
</html>
```
上面的例子实际运行的效果是,当dom节点在解析的过程中,界面上会弹出类似"数据正在加载中",parser解析完毕之后,遮罩层就消失，正常显示页面内（弹出数据加载完成弹出框）。

#Parser(解析器)应用场景
上面的学习中我们知道,easyui根据class就能正常的渲染页面都是靠parser。通常情况下我们在开发的时候并不会用到解析器。下面来看看神马时候我们需要用到解析器。
##自动调用parser
最主要的运用场景,只要我们书写相应的class,easyui就能成功的渲染页面,这是因为解析器在默认情况下,会在dom加载完成的时候($(docunment).ready)被调用,而且是渲染整个页面。
##手动调用parser
需要手动调用的情况是:当页面已经加载完成,但是此时我们使用js生成的DOM中包含了easyui支持的class,并且我们也有将其渲染成easyui组件的需求。在这种情况下手动调用parser就不可避免了。
以如下代码为例:
```bash
<div class="easyui-accordion" id="tt">
    <div title="title1">1</div>
	<div title="title2">2</div>
</div>
```
当上述代码的生成在easyui渲染界面之后,由于easyui不会一直监听页面,所以该段代码并不会并渲染成“手风琴DIV”的样式,这时候就需要我们手动去结下了。不过这里需注意如下方面，上面也有提及。
    1. 解析目标位指定DOM的所有子孙元素,不包好该DOM本身:因此正确的写法为:$parser.parser($('tt').parent()),并非
       $.parser.parse($('#tt'));    
    2. 尽量不要多次解析同一个DOM元素(ID):页面初始化就已经主动渲染过dom节点了,你再次手动解析该dom节点时该dom已经被parser重构,得到的        DOM就并非是你料想的结果，该方式应该尽量避免。