title: Go语言时间处理
date: 2015-04-15 14:55:24
tags: [golang,time]
categories: golang
---
&nbsp;&nbsp;&nbsp;&nbsp;一次群里面的朋友在问Unix时间戳的转换问题,刚好无聊在写go代码,于是就查询了下time包,实现了时间戳转换为时间。不用说time包开发中基本是必须用到的包之一,因此也就顺便做一个总结了。

&nbsp;&nbsp;&nbsp;&nbsp;查看官方文档,time包里面包含了许多数据类型,不过最常见,使用的最多的必然要属Time了,这个Time类型最小可以表示到nanosecond(微毫秒,十亿分之1秒)。
<!--more-->
<center>![go_time](http://kiritor.github.io/img/go_time.png)</center>
&nbsp;&nbsp;&nbsp;&nbsp;接下来介绍一些比较常用的方法和使用
##时间戳
&nbsp;&nbsp;&nbsp;&nbsp;获取当前的时间戳:
```GO
    t := time.Now().Unix()
    fmt.Println(t)
	//1429081897(单位秒)
```
##格式化时间
&nbsp;&nbsp;&nbsp;&nbsp;最常见的需求,实现方式如下:
```GO
fmt.Println(time.Now().Format("2006-01-02 15:04:05"))  // 这是个奇葩,必须是这个时间点, 据说是go诞生之日, 记忆方法:6-1-2-3-4-5
//2014-01-07 09:42:20
```
##时间戳转换为格式化时间
&nbsp;&nbsp;&nbsp;&nbsp;思路是先将时间戳转化为时间,在转化为格式化字符串
```GO
str_time := time.Unix(1389058332, 0).Format("2006-01-02 15:04:05")
fmt.Println(str_time)
//2014-01-07 09:32:12
```
##格式化字符串转时间
&nbsp;&nbsp;&nbsp;&nbsp;同样是先将字符串转换为时间,在转化为时间戳,两种方法:
&nbsp;&nbsp;&nbsp;&nbsp;方法一:
```GO
the_time := time.Date(2014, 1, 7, 5, 50, 4, 0, time.Local)
unix_time := the_time.Unix()
fmt.Println(unix_time)
//389045004
```
&nbsp;&nbsp;&nbsp;&nbsp;方法二:
```GO
the_time, err := time.Parse("2006-01-02 15:04:05", "2014-01-08 09:04:41")
if err == nil {
        unix_time := the_time.Unix()
    fmt.Println(unix_time)     
}
//1389171881
```
##时间比较
&nbsp;&nbsp;&nbsp;&nbsp;Time的比较时使用Before,After,Equal方法,返回值为bool类型
##输出星期信息
&nbsp;&nbsp;&nbsp;&nbsp;输出星期信息也比较简单,方法如下:
```GO
fmt.Println(time.Now().Weekday())
//Wednesday
```
&nbsp;&nbsp;&nbsp;&nbsp;当然Time包下还有许多其他有用的类型例如(Ticker、Timer等),不过这里就不在介绍了,OK,GO时间处理就先到这个地方了!
