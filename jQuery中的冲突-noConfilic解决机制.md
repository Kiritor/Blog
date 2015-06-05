title: jQuery中的冲突-noConfilic解决机制
date: 2015-04-02 14:24:04
tags: noConfilic
categories: front
---
许多的JS框架类库都选择使用符号作为函数或变量名,而且在实际的项目开发中,使用模板语言的话有可能符号即为该模板语言的关键字。例如Veclocity模板语言,是关键字.与jQuery一起使用可能会存在冲突(页面中直接写jq代码,引入的js文件不存在该问题)。吐槽下为啥这么多js库喜欢用( is money?)。

   jQuery是使用符号作为函数或变量名最为典型的一个。在jQuery中,符号只是window.jQuery对象的一个引用,因此即使被删除,jQuery依然能保证整个类库的完整性。
<!--more-->

   jQuery的设计充分考虑了多框架之间的引用冲突。我们可以使用jQuery.noConflict方法来轻松实现控制权的转交。

   在论述如何解决jQuery冲突之前,我们有必要先对noConflict函数做一个了解,解决冲突的方法就藏在里面。
jQuery.noConflict

jQuery.noConflict([removeAll]);

   缺省参数情况下:

   运行这个函数将变量$的控制权让渡给第一个实现它的库。在运行完这个函数之后,就只能使用jQuery变量访问jQuery对象(函数不带参数),例如jQuery("div p")。不过需要注意的是该函数必须在你导入jQuery文件之后,并且在导入另一个导致冲突的库"之前"使用。当然也应该在其他冲突库被使用之前,除非jQuery是最后一个导入的。

   当参数为true时,执行noConflict则会将$和jQuery对象本省的控制权全部移交给第一个产生他们的库。

   不过具体的移交机制是如何实现的呢?查阅源码即可发现,在jQuery源码中定义了两个私有变量_jQuery,_。具体如下截图:

   <center>![](http://liangtao-wordpress.stor.sinaapp.com/uploads/2014/06/QQ截图20140618134225.png)</center>

   容易理解的是,jQuery通过上面两个私有的变量映射了window环境下的jQuery和$两个对象,防止了变量被强行覆盖。一旦noConflict被调用,jquery可以通过_jQuery,_,jQuery,四者之间的差异,来决定控制权的移交方式,具体代码如下图:

   <center>![](http://liangtao-wordpress.stor.sinaapp.com/uploads/2014/06/QQ截图20140618134652.png)</center>

   接下来看看参数设定问题,如果deep没有设置,_覆盖了window,此时jQuery的别名失效了,但是jQuery变量未失效,仍可使用。此时如果有其他库或代码重新定义了$变量的话,其控制权就转交出去了。反之deep设置为true时，_jQuery进一步覆盖window.jQuery,此时和jQuery都将失效。

   这种操作的好处是,不管是框架混用还是jQuery多版本共存这种高度冲突的执行环境,由于noConflict的控制权移交机制,以及本身返回违背覆盖的私有变量jQuery对象,完全能够通过变量映射的方式解决冲突。
示例

   了解了jQuery内部解决冲突的实现方式,接下来看看一些实际的情况吧。

   1、将$引用的对象映射回原始的对象。
```bash
jQuery.noConflict();
// 使用 jQuery
jQuery("div p").hide();
// 使用其他库的 $()
$("content").style.display = 'none';
```
   2、恢复使用别名$,然后创建并执行一个函数,在这个函数的作用域中仍然将$作为jQuery的别名来使用。在这个函数中,原来的$对象是无效的。这个函数对于大多数不依赖于其他库的插件都十分有效。
```bash
jQuery.noConflict();
(function($) { 
  $(function() {
    // 使用 $ 作为 jQuery 别名的代码
  });
})(jQuery);
// 其他用 $ 作为别名的库的代码
```
   3、创建一个新的别名用来在接下来的库中使用jQuery对象。
```bash
var j = jQuery.noConflict();
// 基于 jQuery 的代码
j("div p").hide();
// 基于其他库的 $() 代码
$("content").style.display = 'none';
```
   基于这种方式,所有的jQuery代码都通过j进行调用,避免了冲突的可能。

   4、完全将jQuery移到一个新的命名空间。
```bash
var dom = {};
dom.query = jQuery.noConflict(true);
```
   结果:
```bash
// 新 jQuery 的代码
dom.query("div p").hide();
// 另一个库 $() 的代码
$("content").style.display = 'none';
// 另一个版本 jQuery 的代码
jQuery("div > p").hide();
```
#解决冲突方式

 冲突的方式无非3中情况：
 1、其他库先于jQuery引用(被占用).
  最简单的我们可以在任何地方调用jQuery.noConflict函数,之后使用jQuery()座位jQuery对象的制造工厂。
```bash
 jQuery.noConflict();       //将变量$的控制权移交给先导入的库
	 jQuery(function(){
		 jQuery("p").click(function(){   //使用jQuery变量 
			 
		 });
 });
 $("pp").style.display='none';   //其他库的调用
 ```

 此外,如果你想确保jQuery不会与其他库冲突,但又想使用一个类似"$"的快捷方式,可以使用如下代码:
```bash
 var $mj=jQuery.noConflict();       //自定义快捷变量
	 $mj(function(){
		 $mj("p").click(function(){   //使用jQuery变量 
			 
		 });
 });
 $("pp").style.display='none';   //其他库的调用
 ```
如果你不想给jQuery自定义名称,却想使用$，有不与其他库冲突.可以有两种解决方式:
 其一:
```bash
 jQuery.noConflict();       
 jQuery(function($){
   $("p").click(function(){   //在函数内部继续使用
			 
   });
 });
 $("pp").style.display='none';   //其他库的调用
 ```
其二:
```bash
jQuery.noConflict();       //将变量$控制权让渡
(function($){               //定义匿名函数形参为$
	$(function(){      //匿名函数内部均为jQuery的$
           $("p").click(function(){
           
           });
        });
})(jQuery);                //执行匿名函数且传递实参jQuery
 $("pp").style.display='none';   //其他库的调用
 ```

 这是较为理想的方式,因为可以改变最少的代码来实现全面的兼容性
 2、其他库后于jQuery被引用
你可以参考上述做一些冲突解决的方法,其实其根本就不冲突,你可以使用jQuery变量做一些jQuery的处理工作。同时可以使用()方法作为其他库的快捷方式。
 3、不同版本jQuery、且有其他库
可以参考上述示例,将jQuery完全移到另一个命名空间。
```bash
	var dom = {};
	dom.query = jQuery.noConflict(true);
	// 新 jQuery 的代码
	dom.query("div p").hide();
	// 另一个库 $() 的代码
	$("content").style.display = 'none';
	// 另一个版本 jQuery 的代码
	jQuery("div > p").hide();
```
 #最后
   jQuery解决冲突的机制是十分灵活的,有了这些冲突解决方案,就可以在项目中安心的使用jQuery了。

