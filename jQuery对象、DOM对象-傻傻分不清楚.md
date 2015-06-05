title: jQuery对象、DOM对象?傻傻分不清楚
date: 2015-04-02 13:40:07
tags: DOM
categories: front
---
   初学jQuery时,经常分辨不清楚哪些是jQuery对象,哪些是DOM对象。这是十分不好的现象。必须明确区分何为jQuery对象、何为DOM对象,对于后续的学习、理解才更方便。

   先从DOM对象开始,之后在谈谈jQuery对象(jq对象基于DOM对象)。
#DOM、DOM对象
   DOM(Document Object Model,文档对象模型),DOM是W3C的标准。定义了访问HTML和XML文档的标准。
文档对象模型是中立于平台和语言的接口,允许程序和脚本动态的访问和更新文档的内容、结构以及样式,更具体来说就是我们可以通过js、jQuery代码动态的更新某个html元素的样式、属性等。
<!--more-->
   W3C DOM标准分为3部分:
   1. 核心DOM-针对任何结构化文档的标准模型。
   2. XML DOM-针对XML文档的标准模型。
   3. HTML DOM-针对HTML文档的标准模型。
   这里我们关注的是HTML DOM以及其是如何获取、修改、添加或删除html元素(DOM对象)的。
DOM节点       
   根据W3C的HTML DOM标准,HTML文档中的所有内容都是节点:
   1、整个文档是一个文档节点
   2、每个HTML元素是一个元素节点
   3、每个HTML属性是一个属性节点
   4、注释是注释节点
#HTML DOM节点树
   HTML DOM将HTML文档视作树结构。这种结构成为节点树:
   一个实例：
   <center>![Dom树](http://liangtao-wordpress.stor.sinaapp.com/uploads/2014/06/ct_htmltree.gif)</center>
   关于DOM的介绍就这么多了,详细了解的话可查阅W3CSCHOOL手册。
#DOM对象

   简单的来说通过JavaScript中的getElementsByTagName或者getElementById来获取元素节点,得到的DOM元素就是DOM对象。DOM对象可以使用JavaScript中的方法,简单实例:
```bash
var obj = document.getElementById("id");//获取DOM对象
var ObjHTML = obj.innerHTML;                //使用js方法
```
#jQuery对象

   jQuery对象就是通过jQuery包装DOM对象所产生的对象。

   需注意的是jQuery对象是jQuery独有的。如果一个对象是jQuery对象,那么就可以使用jQuery里的方法。

   一个简单的例子:
```bash
$("#foo").html();   //获取id为foo的元素内的html代码.html()为jQuery里的方法
```
   该代码等价于:
```bash
document.getElementById("id").innerHTML
```

#相互转换

   核心提示:jQuery选择器得到的jquery对象和标准的DOM对象是两种不同的对象,jquery对象不能使用DOM对象的属性方法。同样的DOM对象也不能使用jquery对象的方法、属性。

   不过前面了解到,jquery对象本来就是DOM对象包装而来的,那么两者肯定是能够进行转换的。

   在讨论jQuery对象和DOM对象的相互转换之前,先约定好定义变量的风格。jquery对象在前面加上$，例如:
```bash
var $test = jQuery 对象;        //jquery变量
var test = DOM 对象;           //dom变量
```

##jQuery对象->DOM对象

   jQuery对象是不能使用DOM中的方法的,但是如果对jQuery对象的方法不熟悉,或者jQuery对象没有封装想要的方法,不得不使用DOM对象的时候,可以通过以下两种方式将jQuery对象转换为DOM对象。

   (1) jQuery对象是一个数组对象,可以通过[index]得到相应的DOM对象。
```bash
var $vars = $("#vars");           //jQuery对象
var test = $vars[0];               //DOM对象
console.info(test.checked);    //检查该对象是否选中
```
   (2) 通过jQuery本身提供的get(index)方法返回DOM对象。
```bash
var $vars = $("#vars");           //jQuery对象
var test = $vars.get(0);             //DOM对象
console.info(test.checked);    //检查该对象是否选中
```
##DOM对象->jQuery对象

   对于一个DOM对象,只需要用$()将DOM对象包装起来就可以转换为jQuery对象了。
```bash
var cr = document.getElementById("id");     //DOM对象
var $cr = $(cr);                            //jQuery对象
```
   转换后,可以任意使用jQuery中的方法。

   通过以上方法可以实现DOM对象和jQuery对象的相互转换,但是一般情况下jQuery对象提供了一套更加完善的工具用于操作DOM,因此优选使用jQuery对象操作DOM是一个不错的选择。
示例
```bash
<html>
<head>
    <title>jQuery对象和DOM对象</title>
    <script src="js/jquery-1.9.1.js"></script>
    <style type="text/css">
        body {
            text-align: center;
        }
    </style>
    <script>
		/*使用原生js代码判断复选框是否被选中*/
		$(function(){
			var $cr=$("#cr");
			var cr = $cr.get(0);
			$cr.on('click',function(){
				if(cr.checked){
					console.info("感谢您的支持,你可以继续操作!");
				}
			});
		});
		/*使用jQuery的方式判断复选框是否被选中*/
		$(function(){
		   var $cr = $("#cr");
			$cr.on('click',function(){
				if($cr.is(":checked")){
					alert("通过jQuery方式选中");
				}
			});
		});
    </script>
</head>

<body>
    <input type="checkbox" id="cr"/>
    <label for="cr">我同意该服务条款</label>
</body>
</html>
```
   控制台输出js选中,并弹出jquery选中的信息。
