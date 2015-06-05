layout: post
title: 自己动手开发jQuery插件
category: front
tags: [jQuery,plugin]
---
&nbsp;&nbsp;&nbsp;&nbsp;jQuery的使用越来越广泛,现如今的前端框架基本上都使用了jQuery。jQuery凭借其简洁的API,对DOM强大的操控性,易于扩展,开源,社区化的模式越来越受到开发人员的喜爱。jQuery的迅速流行不仅仅是因为jQuery本身的优势,其易于扩展的插件,开源的社区(良好的生态社区)使得jQuery插件越来越丰富稳定极大的简化了web开发人员的工作,这也是jQuery最为成功的地方。
&nbsp;&nbsp;&nbsp;&nbsp;jQuery的使用并不算复杂,开发中肯定接触了不少的插件.不过,我们不能仅仅只能使用工具,还要学会如何去编写自己的工具:jQuery插件。
<!--more-->
##插件基础
&nbsp;&nbsp;&nbsp;&nbsp;在最基础的水平上,创建jQuery插件是非常简单的,但是需要明白的是:简单并不意味正确,遵循一些最佳实践能够改善性能,减少错误,提升交互性,参考jQuery编码规范:[编写更好的jQuery的建议](http://kiritor.github.io/2015/04/02/%E7%BC%96%E5%86%99%E6%9B%B4%E5%A5%BD%E7%9A%84jQuery%E7%9A%84%E5%BB%BA%E8%AE%AE-%E8%BD%AC/)
###命名规范
&nbsp;&nbsp;&nbsp;&nbsp;一般情况下,插件(开发环境)的命名形式为:
```bash
		jquery.pluginName.js
```
&nbsp;&nbsp;&nbsp;&nbsp;生产环境下(压缩之后的),插件的命名形式为:
```bash
		jquery.pluginName.min.js
```
##jQuery插件开发模式
&nbsp;&nbsp;&nbsp;&nbsp;jQuery高级编程中提到了3中插件开发模式:
&nbsp;&nbsp;&nbsp;&nbsp;1. 通过jQuery.fn向jQuery添加新方法.
&nbsp;&nbsp;&nbsp;&nbsp;2. 通过jQuery.extend()扩展jQuery.
&nbsp;&nbsp;&nbsp;&nbsp;3. 通过jQuery.widget()应用jQueryUI的部件工厂方法创建.
&nbsp;&nbsp;&nbsp;&nbsp;先从简单的第二种方法开始,这种插件开发的方式仅仅是在jQuery命名空间(jQuery)上添加了一个静态方法,可以直接通过美元符号(jquery编码的符号,页面解析不出来,用"美元符号代替")调用,一个简单的例子:
```javascript
$.extend({
   log:function(text) {
      console.log("日志:"+text)
   }
});
//直接通过$符号进行调用
$.log("简单的日志插件")
```
&nbsp;&nbsp;&nbsp;&nbsp;jquery最令人惊叹的便是强大的选择器了,显然这种方式无法利用jquery的选择器,而且这并非是一个规范的做法。jquery.extend用于扩展自身方法,而jquery.fn则是用于扩展jquery类,包括了对方法和对象的操作。为了保持jquery的完整性,推荐使用第一种方法进行插件开发,尽量不要使用第二种。至于第三种方式,则是用于开发更高级jquery部件的,这里就不再细说了,下面重点总结下第一种方式如何进行插件的开发。
##插件开发-雏形
1、开始
&nbsp;&nbsp;&nbsp;&nbsp;根据开篇的命名规范,最初的插件原型可能如下:
```javascript
jQuery.fn.toolTip = function(){
   //do something
};
```
&nbsp;&nbsp;&nbsp;&nbsp;但是为了确保你的插件与其他插件的美元符号不冲突,最好使用一个立即执行的匿名函数,这样就可以放心的使用美元符号了。结构如下:
```javascript
;//可以事先加一个";",这是一个好习惯,可以避免与别人的函数相连,造成报错
(function( $ ){  
	
  $.fn.toolTip = function() {  
    //do something
  };  
	
})( jQuery );  
```
&nbsp;&nbsp;&nbsp;&nbsp;jQuery高级编程中提及了一种更加安全、结构良好、有序的方式(笔者认为上面的方式已经没问题了)供参考:
```javascript
;(function($, window, document,undefined) {
   
    $.fn.toolTip = function(options) {
       
    }
	
})(jQuery, window, document);
```
2、基本开发
&nbsp;&nbsp;&nbsp;&nbsp;准备工作做的差不多了,至少来写一个简单的插件了吧。
```javascript
/**
   给某个元素提示信息
*/

(function( $ ){  
	
  $.fn.toolTip = function() {  
      var $tip = $('<span>'+this.text()+'</span>');
	  $tip.css({
	      color:'red',
		  fontSize:'10px'
	  });
	  this.append($tip);
	  
  };  
	
})( jQuery );  
//调用
$('#id').toolTip();
```
&nbsp;&nbsp;&nbsp;&nbsp;元素调用该插件之后,后面会跟一个提示信息(元素的text信息),字体颜色为红色,size为10px。插件已经初具雏形了!
##插件形态二(默认值和选项)
&nbsp;&nbsp;&nbsp;&nbsp;上面的事例仅仅是一个插件的雏形,远没有达到插件所要求的那样,不够灵活,不易扩展(调整字体的大小，颜色必须修改插件源码)。
&nbsp;&nbsp;&nbsp;&nbsp;为了一些复杂的,可定制化的插件,合理的做法是提供一套默认值,在被调用的时候扩展默认值。这样在实际使用插件时,根据不同的时候配置不同的参数,合理、方便。接上面的插件,实现插件的使用者,自己定义tooltip字体的颜色和大小,可以这样做:
```javascript
/**
   给某个空间提示信息
*/

(function($) {

    $.fn.toolTip = function(options) {

        var settings = $.extend({}, $.fn.toolTip.defaults, options);
        var $tip = $('<span>' + this.text() + '</span>');
        $tip.css({
            color: settings.color,
            fontSize: settings.fontSize
        });
        this.append($tip);

    };

    //插件默认参数
    $.fn.toolTip.defaults = {
        color: 'red',
        fontSize: '8px'
    };

})(jQuery);

//调用
$('#toolTip').toolTip({
     color: '#000',
     fontSize: '30px'
});
```
&nbsp;&nbsp;&nbsp;&nbsp;这样一来的话,就可以通过注入不同的参数来使用插件了,插件真正的做到了灵活扩展,不过离一个强大的插件还不够!
##插件形态三(方法、事件、数据)
&nbsp;&nbsp;&nbsp;&nbsp;在此之前先理解一下命名空间,正确的命名空间对于插件开发是十分重要的,这样能确保你的插件不被其他插件重写,也能够避免被页面上其他代码重写。命名空间记录了插件的方法、事件和数据。
##A、插件方法
&nbsp;&nbsp;&nbsp;&nbsp;千万不在一个插件中为jquery.fn增加多个方法,错误的做法:
```javascript
(function( $ ){  
  
  $.fn.tooltip = function( options ) { };  
  $.fn.tooltipShow = function( ) {   };  
  $.fn.tooltipHide = function( ) {   };  
  $.fn.tooltipUpdate = function( content ) {   };  
  
})( jQuery );  
```
污染了命名空间,可以把所有的方法放在一个对象中,通过参数来调用,正确的姿势如下:(改进上述代码)
```javascript
/**
   给某个控件提示信息
*/

(function($) {

	
	//插件初始化
	function init(target,options){
	    var settings = $.extend({}, $.fn.toolTip.defaults, options);
        var $tip = $('<span>' + settings.text+ '</span>');
		var hidden = '';
		if(settings.show==true){
		    hidden='inline-block';
		}else
			hidden='none';
		$tip.css({
            color: settings.color,
            fontSize: settings.fontSize,
			display:hidden
        });
        $(target).append($tip);
	}
	/*
	   显示消息提示
	   param:isShow是否显示
	*/
	function show(target,isShow){
		if(isShow==true)
	       $(target).find('span').css('display','inline-block');
		else
		   $(target).find('span').css('display','none');
	 
	}
	
	
    $.fn.toolTip = function(options,param) {
   
        if(typeof options == 'string'){
		    var method = $.fn.toolTip.methods[options];
			if(method)
				return method(this,param);
		}else 
			init(this,options);

    };

    //插件默认参数
    $.fn.toolTip.defaults = {
        color: 'red',
        fontSize: '8px',
		show: true,
		text:''
    };
	
	//插件方法集合
	
	$.fn.toolTip.methods = {
	    
		init:function(jq,options){
		    return jq.each(function(){
			   init(this,options);
			});
		},
		show:function(jq,isShow){
		    return jq.each(function(){
				show(this,isShow);
			});
		}
	 
	}

})(jQuery);
//页面进行调用
$('#toolTip').toolTip({
    color: 'red',
    fontSize: '6px',
	text:'我是提示消息'
});
$('#toolTip').toolTip('show',false)
```
&nbsp;&nbsp;&nbsp;&nbsp;实际情况是:提示信息不在显示了,ok!到此为止就可以给自己的插件添加各种有意思的方法了,插件也是越来越强大了呢!
##B、插件事件
&nbsp;&nbsp;&nbsp;&nbsp;这里所说的事件和传统的事件略有区别。考虑这样一种需求:当一个插件完成某种操作之后,需要一些自定义的功能。把这一过程叫做事件,这里和浏览器的诸如onclick事件是有很大差异的,更多的是插件业务逻辑上的延伸,本质上是插件完成某件操作事件)之后的回调操作,只是这个操作是用户自己实现。核心实现在于call方法的运用。
&nbsp;&nbsp;&nbsp;&nbsp;还是以上面的插件为例子,我们给插件定义一个AfterInit事件,当插件初始化完成之后,用户可以自定义一些操作。这只是操作逻辑上看起来像一个事件,其实本质上是一个空方法(再插件内部调用),代码如下:
```javascript
//插件方法集合
	
	$.fn.toolTip.methods = {
	    
		init:function(jq,options){
		    return jq.each(function(){
			   init(this,options);
			});
		},
		show:function(jq,isShow){
		    return jq.each(function(){
				show(this,isShow);
			});
		},
		//定义一个事件(方法),实现为空
		AfterInit:function(){}
	}
	//
	//插件初始化
	function init(target,options){
	    var settings = $.extend({}, $.fn.toolTip.defaults, options);
        var $tip = $('<span>' + settings.text+ '</span>');
		var hidden = '';
		if(settings.show==true){
		    hidden='inline-block';
		}else
			hidden='none';
		$tip.css({
            color: settings.color,
            fontSize: settings.fontSize,
			display:hidden
        });
        $(target).append($tip);
		options.AfterInit.call(target);   //初始化完成之后,触发事件(本质上就是个方法)
	}
```
本质上就是个方法，只是用起来像一个事件了,接下来,我们在使用中自己实现回调事件的操作:
```javascript
  $('#toolTip').toolTip({
        color: 'red',
        fontSize: '6px',
	    text:'我是提示消息',
		AfterInit:function(){
		   alert("插件已经初始化完成!")
	    }
   });
```
这样在插件初始化完成之后,就会触发AfterInit事件(方法),完成用户自定义的操作。这样以来,插件的功能更加强大了呢。
##B、插件数据
&nbsp;&nbsp;&nbsp;&nbsp;在插件开发的时候我们需要在插件中保存设置和信息,这是jquery的data方法就非常有用了,他会获取元素的相关数据,如果数据不存在,创建相应的数据添加到元素上。下面看看如何使用吧:
```javascript
//插件初始化
	function init(target,options){
	    var settings = $.extend({}, $.fn.toolTip.defaults, options);
		$this = $(target);
		//保存设置
		$this.data('toolTip');
        var $tip = $('<span>' + settings.text+ '</span>');
		var hidden = '';
		if(settings.show==true){
		    hidden='inline-block';
		}else
			hidden='none';
		$tip.css({
            color: settings.color,
            fontSize: settings.fontSize,
			display:hidden
        });
        $this.append($tip);
		options.AfterInit.call(target);   //初始化完成之后,触发事件(本质上就是个方法)
	}
	//插件销毁
	function destory(target){
	    $(target).removeData('toolTip');  //删除数据
	}
```
使用data方法可以帮助你在插件的各个方法间保持状态和变量,将之放在一个对象中,便于访问,也便于删除。
##插件支持链式调用
&nbsp;&nbsp;&nbsp;&nbsp;我们知道jquery的一个优雅的设计是支持链式操作的,即使你不准备为你的插件提供链式支持,但是为这准备也是一个很好的实践,上面的插件其实已经实现了,关键在于每次都要return插件本身(this)。
```javascript
show:function(jq,isShow){
    return jq.each(function(){
		show(this,isShow);
	});
}
//链式调用
$('#toolTip').toolTip({
    color: 'red',
    fontSize: '6px',
    text:'我是提示消息',
	AfterInit:function(){
	    alert("插件已经初始化完成!")
	   }
    });
$('#toolTip').toolTip('show',true).css('color','red');
```
##最佳实践
&nbsp;&nbsp;&nbsp;&nbsp;下面是编写jquery插件的一些总结与最佳实践。
－ 使用(function($){//plugin})(jQuery);来包装你的插件
－ 返回this指针保证插件可链式操作
－ 使用对象来设置差价的参数和默认值
－ 为你的函数、事件、数据附着到某个命名空间
##代码压缩
&nbsp;&nbsp;&nbsp;&nbsp;代码混淆与压缩不仅能一定程度上保护代码,更是使得插件的体积减小,变得轻量级,加快了下载速度。
##分享
&nbsp;&nbsp;&nbsp;&nbsp;最后,好的插件要分享出来哦,那样插件才具有生命力,才能不断的优化改进!
##相关参考
[插件开发精品教程](http://www.cnblogs.com/Wayou/p/jquery_plugin_tutorial.html)
[jquery插件开发](http://www.bkjia.com/jQuery/963539.html)