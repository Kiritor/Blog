title: 编写更好的jQuery的建议-转
date: 2015-04-02 11:35:38
tags: jQuery
categories: front
---
   最近学习JQuery,在[伯乐在线](http://blog.jobbole.com/)里面看到了一片非常不错的翻译文章,觉得对于新手来说非常实用,打算转载过来,自己也略作了一些修改,例如链式操作。
   文章出处:[http://blog.jobbole.com/52770/](http://blog.jobbole.com/52770/)
   译文地址:[原文](http://flippinawesome.org/2013/11/25/writing-better-jquery-code/)
   以下为翻译正文:

讨论jQuery和javascript性能的文章并不罕见。然而,本文我计划总结一些速度方面的技巧和我本人的一些建议,来提升你的jQuery和javascript代码。好的代码会带来速度的提升。快速渲染和响应意味着更好的用户体验。

   首先,在脑子里牢牢记住jQuery就是javascript。这意味着我们应该采用相同的编码惯例,风格指南和最佳实践。
   如果你是一个javascript新手,我建议您阅读《JavaScript初学者的最佳实践》,这是一篇高质量的javascript教程,接触jQuery之前最后先阅读。
   当你准备使用jQuery,我强烈建议你遵循下面这些指南:
<!--more-->
#缓存变量

   要知道DOM的遍历是昂贵的,所以尽量将会重用的元素缓存到一个变量中。
```bash
// 糟糕
h = $('#element').height();
$('#element').css('height',h-20);
// 建议
$element = $('#element');
h = $element.height();
$element.css('height',h-20);
```
#避免全局变量
jQuery与javascript一样,一般来说,最好确保你的变量在函数作用域内。
```bash
// 糟糕
 
$element = $('#element');
h = $element.height();
$element.css('height',h-20);
 
// 建议
 
var $element = $('#element');
var h = $element.height();
$element.css('height',h-20);
```
#使用匈牙利命名法
在变量前加$前缀,便于识别出jQuery对象。
```bash
// 糟糕
 
var first = $('#first');
var second = $('#second');
var value = $first.val();
 
// 建议 - 在jQuery对象前加$前缀
 
var $first = $('#first');
var $second = $('#second'),
var value = $first.val();
```

#使用Var链(单Var模式)
将多条var语句合并为一条语句,我建议将未复制的变量放到后面。
```bash
var
  $first = $('#first'),
  $second = $('#second'),
  value = $first.val(),
  k = 3,
  cookiestring = 'SOMECOOKIESPLEASE',
  i,
  j,
  myArray = {};
  ```

#'on'附加事件
在新版jQuery中,更短的on('click')用来取代类似click()这样的函数。在之前的版本中on()就是bind()。自从jQuery1.7版本后,on()?附加事件的方式为处理程序的首选方式。然而,出于一致性考虑,你可以简单的全部使用on()方法。
```bash
// 糟糕
 
$first.click(function(){
    $first.css('border','1px solid red');
    $first.css('color','blue');
});
 
$first.hover(function(){
    $first.css('border','1px solid red');
})
 
// 建议
$first.on('click',function(){
    $first.css('border','1px solid red');
    $first.css('color','blue');
})
 
$first.on('hover',function(){
    $first.css('border','1px solid red');
})
```

#精简代码

一般来说,最好尽可能合并函数。
```bash
// 糟糕
 
$first.click(function(){
    $first.css('border','1px solid red');
    $first.css('color','blue');
});
 
// 建议
 
$first.on('click',function(){
    $first.css({
        'border':'1px solid red',
        'color':'blue'
    });
});
```

#链式操作
jQuery支持非常优雅的链式编码风格,不过需要注意的是进行链式编码的时候可能带来代码的难以阅读,添加缩进和换行来提高代码可读性是十分有必要的。
```bash
// 糟糕
 
$second.html(value);
$second.on('click',function(){
    alert('hello everybody');
});
$second.fadeIn('slow');
$second.animate({height:'120px'},500);
 
// 建议
 
$second.html(value);
$second.on('click',function(){
    alert('hello everybody');
}).fadeIn('slow').animate({height:'120px'},500);
```

上述代码可读性也不是太好,我们可以考虑添加一条链就换行,如下:
```bash
 $(function() {
     //链式操作的时候添加紧缩和换行提高代码的可读性。
     $(".has_children").on('click', (function() {
                $(this).addClass("highlight") //点击该元素的时候高亮
                .children("a")
                 .show()
                .end() //显示该元素的子元素
                .siblings()
                .removeClass("highlight") //获取兄弟元素并去掉其高亮(实际效果就是之高亮一个)
             .children("a")
        .hide(); //隐藏兄弟元素的子元素
    }));
  });
```
 上述代码相较于之前,层次更加清晰了,可读性加强。不过如果你认为代码行数过多的话,可以考虑以功能块来进行换行。例如上述代码可改写成:
```bash
   $(function() {
            //链式操作的时候添加紧缩和换行提高代码的可读性。
            $(".has_children").on('click', (function() {
                $(this).addClass("highlight") //点击该元素的时候高亮
                .children("a").show().end() //显示该元素的子元素
                .siblings().removeClass("highlight") //获取兄弟元素并去掉其高亮(实际效果就是之高亮一个)
                .children("a").hide(); //隐藏兄弟元素的子元素
            }));
        });
```

#选择短路求值
短路求值是一个从左到右求值的表达式,用$$或||操作符。如果追求极致性能的话,你甚至可以考虑&和|操作符。
```bash
// 糟糕
 
function initVar($myVar) {
    if(!$myVar) {
        $myVar = $('#selector');
    }
}
 
// 建议
 
function initVar($myVar) {
    $myVar = $myVar || $('#selector');
}
```

#选择捷径
精简代码的其中一种方式便是利用编码的捷径。

```bash
// 糟糕(其实也算不上糟糕啦)
 
if(collection.length > 0){..}
 
// 建议
 
if(collection.length){..}

繁重的操作中分离元素

      如果你打算对DOM元素做大量操作(连续设置多个属性或css样式),建议首先分离元素然后在添加。

// 糟糕
 
var
    $container = $("#container"),
    $containerLi = $("#container li"),
    $element = null;
 
$element = $containerLi.first(); 
//... 许多复杂的操作
 
// better
 
var
    $container = $("#container"),
    $containerLi = $container.find("li"),
    $element = null;
 
$element = $containerLi.first().detach(); 
//... 许多复杂的操作
 
$container.append($element);
```

#熟记技巧
你可能对使用jQuery中的方法缺少经验,一定要查看文档,因为可能会有一个更好或更快的方法来使用,当然,这个过程需要积累。
```bash
// 糟糕
 
$('#id').data(key,value);
 
// 建议 (高效)
 
$.data('#id',key,value);
```

#使用子查询缓存的父元素
jQuery最便利的就是查询了,不过正如前面所提到了,DOM遍历查询是一项昂贵的操作。典型的做法是缓存父元素并在选择子元素时重用这些缓存元素。
```bash
// 糟糕
 
var
    $container = $('#container'),
    $containerLi = $('#container li'),
    $containerLiSpan = $('#container li span');
 
// 建议 (高效)
 
var
    $container = $('#container '),
    $containerLi = $container.find('li'),
    $containerLiSpan= $containerLi.find('span');
```

#避免通用选择符
将通用选择符放到后代选择符中,性能是非常糟糕的。
```bash
// 糟糕
 
$('.container > *'); 
 
// 建议
 
$('.container').children();
```

#避免隐式通用选择符
通用选择符有时是隐式的,不容易被发现,后续可能会造成严重后果。
```bash
// 糟糕
 
$('.someclass :radio'); 
 
// 建议
 
$('.someclass input:radio');
```

#优化选择符

例如,id选择符是唯一的,所以没有必要画蛇添足的添加额外的选择符。
```bash
// 糟糕
 
$('div#myid'); 
$('div#footer a.myLink');
 
// 建议
$('#myid');
$('#footer .myLink');
```

#避免多个ID选择符

在此强调,id选择符是唯一的,不需要添加额外的选择符,更不需要多个后代ID选择符。
```bash
// 糟糕
 
$('#outer #inner'); 
 
// 建议
 
$('#inner
```

#坚持最新版本
新版本的通常更好:更轻量级,更高效。显然,你需要考虑你要支持的代码的兼容性。例如,2.0版本不支持ie 6/7/8。
摒弃弃用方法，关注每个新版本的废弃方法是非常重要的并尽量避免使用这些方法。
```bash
// 糟糕 - live 已经废弃
 
$('#stuff').live('click', function() {
  console.log('hooray');
});
 
// 建议
$('#stuff').on('click', function() {
  console.log('hooray');
});
// 注：此处可能不当，应为live能实现实时绑定，delegate或许更合适
```

#利用CDN
   谷歌的CDN能保证选择离用户最近的缓存并迅速响应。(使用谷歌CDN请自行搜索地址,此处地址已不能使用,推荐使用jquery官网提供的CDN地址)
组合jQuery和javascript原生代码

   如上所述,jQuery就是javascript,这意味着使用jquery能做的事,同样可以用原生代码来做。原生代码的可读性和可维护性可能不如jquery，而且代码量更大。但是也意味着高效(通常更接近底层代码可读性越差,性能越高)。牢记没有任何框架能比原生代码更小、更轻、更高效。

   鉴于vanilla和jQuery之间的性能差异,我强烈建议吸收两人的精华,使用(可能的话)和JQuery等价的原生代码。
最后忠告

   最后,我记录这篇文章的目的是提高jQuery性能和其他一些好的建议。如果你想深入研究对这个话题你会发现很多乐趣。记住,jQuery并非不可或缺,仅仅是一种选择。思考为什么使用它。DOM操作?ajax?模板?css动画?还是选择符引擎?或许javascript微型框架或jQuery的定制版是更好的选择,又或许大道至简,最终选择原生js也说不准呢。
