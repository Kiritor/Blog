layout: post
title: Go语言数组&数组切片整理
date: 2015-06-04 19:41:31
category: golang
tags: [slice,append]
---
# 介绍
&#160; &#160; &#160; &#160;**<span class="color">数组</span>**是编程语言中最常用的功能之一,顾名思义,数组就是指一系列同一类型数据的集合.数组是很有价值的数据结构,因为它的内存分配是连续的,意味着迭代和移动非常迅速.数组看起来是比较简单的,但是一个数组的设计核心的几个问题需要解决,如::

&#160; &#160; &#160; &#160;* **固定大小或可变大小?**
&#160; &#160; &#160; &#160;* **是类型的一部分?**
&#160; &#160; &#160; &#160;* **多维数组的模型?**
&#160; &#160; &#160; &#160;* **空数组的意义**
&#160; &#160; &#160; &#160;* **...**

&#160; &#160; &#160; &#160;这些问题的解决影响着数组仅仅是语言的一个功能还是其设计的核心部分.
<!--more-->
# 数组
&#160; &#160; &#160; &#160;数组是Go语言非常重要的数据结构(**<span class="color">slice</span>**、**<span class="color">map</span>**的底层结构都是基于数组的),长度固定且是值类型:赋值或函数参数调用都将产生一次复制.
## 数组声明和初始化
&#160; &#160; &#160; &#160;数组有如下几种创建方式:
```go
    //声明一个长度为10的int类型的数组
	var array0 [10]int   
	//声明并初始化
	var array1 [5]int = [5]int{1,2,3,4,5}
	//声明并未索引(0开始)为1和4位置指定元素初始化
	//剩余位置为0
	var array2 [5]int = [5]int{1:1,4:5}
	//声明并初始化
	    array3 :=[5]int{}
	//go编译器推导数组长度2
	    array4 :=[...]{1,2}
	//二维数组
	var array5 [2][2]int
```
&#160; &#160; &#160; &#160;一旦数组被声明了,那么他的数据类型和长度就不能在改变了,如果你需要更多的元素,请创建一个新的数组,然后把原有数组的数据拷贝进去.

&#160; &#160; &#160; &#160;Go语言中任何变量被声明时,都会被默认初始化为**各自类型的0值**,数组也是一样的,当一个数组被声明时,它里面的每个元素都会被初始化为0值.
## 数组的容量和长度
&#160; &#160; &#160; &#160;数组的容量和长度是一样的,cap()和len()输出的值一样.
## 数组的使用
&#160; &#160; &#160; &#160;使用[]操作符来访问数组元素:
```go
      array :=[2]int{1,2}
	  array[1] = 1
```
&#160; &#160; &#160; &#160;Go语言中数组是一个**值类型**,所以可以用它来进行赋值操作.一个数组可以被赋值给任意相同类型的数组:
```go
     var array [5]int
	 array1 := [5]int{}
	 array = array1
```
&#160; &#160; &#160; &#160;**Tips:**需要注意的是数组的类型同时包括数组的长度和元素类型,<b>数组类型必须完全相同才能相互赋值</b>,下面的操作是错误的.
```go
     var array [10]int
	 array1 := [5]int{}
	 array = array1
```
&#160; &#160; &#160; &#160;关于上面这一点,《GO语言编程》示例代码也有这一处错误.
&#160; &#160; &#160; &#160;数组是值类型,因此数组可以直接通过<B>==</B>,<b>!=</b>判断,示例如下:
```go
	var array [10]int = [10]int{1,2,3,4,5,6,7,8,9,10}
	var array1 [10]int = [10]int{1,2,3,4,5,6,7,8,9,10}
	flag :=array1==array
	fmt.Println(flag)  //true
```
## 函数中使用数组
&#160; &#160; &#160; &#160;考虑到这一点:GO语言中数组是值类型的,在函数中使用参数传递数组是非常昂贵的行为,如果变量是数组,意味着传递整个数组,当数组过大时,严重的影响了开销.but开发者并不那么的傻,更好的办法是传递指向数组的指针,这样开销会大大降低,但是注意到如果在函数中改变指针指向的值,那么原始数组的值也会改变(可以规避,但是无法规避的是传递的数组的长度必须相等)
&#160; &#160; &#160; &#160;对于上述这个问题,以及开头提出的数组的核心设计问题,幸运的是Go语言的<b><span class="color">slice(切片)</span></b>可以帮我们处理好这些问题.
# Slice(切片)
&#160; &#160; &#160; &#160;Slice(切片),首先需要明白的是slice并不是数组,<b>slice描述了与slice变量本身相隔离的,存储在数组里面的连续部分,切片描述的是一段数组</b>.slice可以按照需要增长和收缩,通过内建的appen、relice方法可以很容易的操作slice,而且slice的底层是基于数组的,所以slice的索引、迭代和垃圾回收性能都十分出色.
&#160; &#160; &#160; &#160;SO,可以简单认为slice是一种"动态数组",它拥有自己的数据结构,就像一个struct,包含3个元数据:
```go
     type slice struct {
	     slice中元素的长度
		 指向底层数组的指针
		 slice的容量
	 }
```
&#160; &#160; &#160; &#160;以上只是一个技术性的猜想.
## 声明&初始化
&#160; &#160; &#160; &#160;GO语言创建slice的方式有很多种,下面依次来看.
&#160; &#160; &#160; &#160;1、 内建函数make:
```go
	//1、make函数创建
	//指定slice长度,这是容量默认为长度
	var slice1 = make([]int,5)
	//同时指定长度3和容量5
	var slice2 = make([]int,3,5)
```
&#160; &#160; &#160; &#160;需要注意的是不允许创建长度大于容量的slice,否则会出现如下编译错误:
```go
     len larger than cap in make([]int)
```
&#160; &#160; &#160; &#160;2、 基于数组创建:
```go
//创建一个int slice
	//长度和容量都是5
	slice :=[]int{1,2,3,4,5}
	//初始化一个100元素的slice
	slice1 :=[]int{99:1}
	//基于数组创建一个slice
	var array [10]int = [10]int{1,2}
	slice2 :=array[:4]
```
## nil&empty slice
&#160; &#160; &#160; &#160;考虑到这种场景：在一个返回slice函数发生异常或者数据库查询返回0个结果.nil slice和empty slice都非常有用:
```go
     //创建nil slice
     var slice []int
	 //创建empty slice
	 slice1 :=make([]int,0)
	 //or
	 slice2 :=[]int{}
```
&#160; &#160; &#160; &#160;而且不管是nil slice还是empty slice,内建函数append,len,cap都不会有影响.
## slice的使用
&#160; &#160; &#160; &#160;slice为一个指定索引的元素赋值和数组完全相同,改变单个元素的值使用[]操作符:
```go
	slice := []int{10, 20, 30, 40, 50}
	slice[1] = 25
```
&#160; &#160; &#160; &#160;slice描述的是一段数组,且这段数组的某个范围是共享的,举个栗子:
```go
	var array [10]int =[10]int{1,2,3,4,5,6,7,8,9,10}
	slice1 :=array[2:8]  //索引2开始,第8位置结束
	slice2 :=array[2:6]
	slice2[3]=999;
	fmt.Println(array)
	fmt.Println(slice1)
	fmt.Println(slice2)
```
&#160; &#160; &#160; &#160;上述代码我们得到了两个slice和一个原始的数组,实际上slice1和slice2都具有指向原始数组的指针,但是slice1和slice2描述的一段数组的范围不同(2：8/VS:2:6),因此3者有一定的重合.情况如下图:
<center>![slice](http://kiritor.github.io/img/golangslice.png)</center>
&#160; &#160; &#160; &#160;因此上述代码的输出结果为:
```bash
	[1 2 3 4 5 999 7 8 9 10]
	[3 4 5 999 7 8]
	[3 4 5 999]
```
&#160; &#160; &#160; &#160;再次强调的是不论是array还是slice1、slice2只要改变的是上述共享数组片段里面的值,都会变化.
&#160; &#160; &#160; &#160;一个slice只能访问它长度范围内的索引,试图访问超出范围的索引将会出现一个运行时错误.
```bash
	Runtime Exception:
	panic: runtime error: index out of range
```
## slice增长
&#160; &#160; &#160; &#160;slice比较数组的优势在于它可以按照我们的需求增长,我们只需要使用append方法,go已经为我们做好了一切.
&#160; &#160; &#160; &#160;append方法需要一个源slice和需要附加到它里面的值,返回一个新的slice,append总是增加slice的长度,另一个方面,如果slice容量足够大,那么底层数组是不会发生改变的,否则会重新分配内存空间.
```go
	// 创建一个长度和容量都为5的 slice
	slice := []int{10, 20, 30, 40, 50}
	// 创建一个新的 slice
	newSlice := slice[1:3]
	// 为新的 slice append 一个值
	newSlice = append(newSlice, 60)
	//slice索引为3的值也变成了60
```
&#160; &#160; &#160; &#160;如果没有足够可用的容量,append函数会创建一个新的底层数组,拷贝已存在的值和将要被附加的新值:
```go
    var array [2]int =[2]int{1,2}
	slice1 :=array[0:]
	slice2 :=array[0:]
	slice2 = append(slice2,60)
	//因为超过了,容量2倍 为4,且不再和slice1、array共享数组片段
	slice2[0] =100
	fmt.Println(array)
	fmt.Println(slice1)
	fmt.Println(slice2)
	fmt.Println(cap(slice2)) //4
	/*
      result:
		[1 2]
		[1 2]
		[100 2 60]
		4
    */
```
&#160; &#160; &#160; &#160;append函数重新创建底层数组时,容量将是现有数组的两倍(<=1000),大于1000之后,容器因子为1.25倍.
##底层数组保护机制
&#160; &#160; &#160; &#160;细心的人可能发现了,append操作很有可能污染了底层数组(当append没有超过容量的时候,底层数组元素被修改了),有时候这并不是我们原意看到的,Go同样帮我们解决了这一问题.通过使用slice第三个索引参数:
```go
    var array [10]int =[10]int{1,2,3,4,5,6,7,8,9,10}
	//第三个参数指定容量
	slice1:= array[2:8:8]
	slice2:= array[2:8]
```
&#160; &#160; &#160; &#160;新建的slice1长度为6,容量也为6,,最大容量为6*2=12计算方法很简单:
```go
	对于 slice[i:j:k] 或者 [2:3:4]
	长度： j - i 或者 3 - 2
	可用容量:  (j - 1)*2
	实际容量： k - i 或者 4 - 2
```
&#160; &#160; &#160; &#160;如果我们试图设置比可用容量更大的容量,会得到一个运行时错误:
```bash
	slice1 := source[2:8:16]
	Runtime Error:
	panic: runtime error: slice bounds out of range
```
&#160; &#160; &#160; &#160;SO,通过第三个索引参数限定,我们可以设置其实际容量和长度相等,这样在append的时候就不会污染底层数组了.内建函数append是变参函数,可以一次添加多个元素.同数组一样另外两个内建函数len、cap返回长度和容量.
##函数间传递slice
&#160; &#160; &#160; &#160;前面就已经知道了,在函数间传递数组是非常昂贵的,使用指针又会出现其他问题,解决的方案就是使用slice.函数间传递slice是非廉价,因为slice相对于是指向底层数组的指针,不过需要注意的是,这里还是会存在污染底层数组的问题(可以规避).