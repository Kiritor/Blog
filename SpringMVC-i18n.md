title: Spring MVC + Velocity实现国际化配置
date: 2015-09-11 18:35:01
tags: [i18n,velocity]
categories: work
---
web开发中,国际化是需要考虑的一个问题,而且这个问题一般是越早敲定越好(不然等到系统大了,翻译是个问题).下面是结合实际项目(Spring MVC+Velocity)对实现国际化的一些总结,项目地址:[https://github.com/kiritor/hr](https://github.com/kiritor/hr)
需要说明的是,该项目使用的是基于Cookie的国际化配置,其他方式参考本文.
##Spring国际化
<b>I18N</b>:作为"国际化"的简称,其来源是英文单词internationalization的首末字符i和n,18为中间的字符数.
<!-- more -->
Spring做国际化的配置主要有3个关键点:
>  1. ResourceBundleMessageSource:实现国际化资源的定义.
>  2. LocaleResolver:实现本地化信息的解析.
>  3. LocaleChangeInterceptor:实现本地化信息的监听(来实现url参数动态指定locale).

###<b>LocaleResolver</b>
LocaleResolver是指用什么策略来检测请求是哪一种locale,Spring MVC提供了一下几种策略:
####<b>AcceptHeaderLocaleResolver</b>
根据浏览器Http Header中的accept-language域判定浏览器的语言环境,可以通过HttpServletRequest.getLocale获得域的内容,但是无法调用LocaleResolver接口的setLocale设置locale.基于这个策略,在后面的demo中可以实现基于浏览器的国际化案例.
####<b>SessionLocaleResolver</b>
根据用户本次会话过程中的语言设定决定语言种类,session级别的,在此session周期内可以修改语言种类,但是session失效后,语言设定失效.基于这个策略,在后面的demo中可以实现基于session的国际化案例.
####<b>CookiedLocaleResolver</b>
根据Cookie判定用于的语言设定(Cookie中保存着用户前一次的语言设定参数).
####<b>FixedLocaleResolver</b>
一直使用固定的Locale,改变locale是不支持的.
如果需要使用哪一种策略,只需要在DispatcherServlet制定的Spring配置文件中配置就行,DispatchServlet将在初始化的时候调用initLocaleResolver(context)方法去配置文件中找名字为localeResolver的bean,如果有就使用配置文件的,没有就使用默认的AcceptHeaderLocaleResovler
通过上面,了解了Spring实现国际化的相关概念,下面结合demo实例,看看Spring MVC是如何实现国际化的.

##基于浏览器请求的国际化
使用AcceptHeaderLocaleResolver策略实现基于浏览器语言的国际化。
1. 首先配置国际化的资源文件.
```xml
<!-- 国际化配置 -->
	<bean id="messageSource"
		class="org.springframework.context.support.ResourceBundleMessageSource">
		<!-- 国际化信息资源文件所在的目录 -->
		<property name="basename" value="i18n.messages" />
		<!-- 找不到对应的代码信息,就用这个代码作为标识 -->
		<property name="useCodeAsDefaultMessage" value="true"/>
	</bean>
```
实例是基于maven项目的,国际化资源文件所在的目录位:java/main/resources/i18n,后缀为.properties:
内容如下:
```xml
#中文
#header
header.message=\u6D88\u606F  
header.to-list=\u5F85\u529E\u4E8B\u5B9C
header.language=\u8BED\u8A00

#英文
#header
header.message=Message
header.to-list=ToDo-List
header.language=Language
```
2. 配置本地化信息解析策略,这里基于浏览器的为默认配置,因此可以不做配置.
基于浏览器当前的语言环境来做控制界面显示的语言,每个URL都是一样的,SpringMVC的AcceptHeaderLocaleResolver都会根据当前浏览器的语言环境,解析对应的信息.
3. 最后前端界面通过使用spring针对不同view视图提供的标记处理国际化信息.jsp方式:
```jsp
<spring:message code="money"/>:<br/>
```
这里我使用的Velocity模板引擎,针对Velocity的标记是:
```velocity
#springMessage("header.message")
```
4. 运行测试:
<center>![中文环境](http://kiritor.github.io/img/i18n_01.png)</center>
可以看出当前浏览器语言顺序为中文优先,更改浏览器语言环境为英文优先,刷新页面:
<center>![英文环境](http://kiritor.github.io/img/i18n_02.png)</center>
到此基于浏览器的国际化配置就完成了,由于是根据浏览器的语言环境来解析的,配置十分的简单.细心的读者可能发现一个问题,前端界面的国际化可以通过配置资源文件解决,那么后端的数据该如何国际化呢?如果只是一些简单的数据,也可以通过配置文件解决,只是压入的view的key-value中的key保持和资源文件一致,实际向view注入对象的时候使用如下方式:
```java
 //从后台代码获取国际化信息
 RequestContext requestContext = new RequestContext(request);
 model.addAttribute("money", requestContext.getMessage("money"));
 model.addAttribute("date", requestContext.getMessage("date"));
```
但是如果数据量过大,例如从数据库的来的数据考虑国际化问题又是另一个层面上的东西了(而且很少会考虑这块的内容,不会考虑用户的输入)
##基于Session的国际化实现
基于Session的国际化配置比基于浏览器的略复杂,在其基础上,需要经过如下配置.
1. 配置本地化信息解析策略:
```xml
	<bean id="localeResolver"
		class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
```
2. SpringMVC配置解析拦截器,我们需要手动切换系统的语言(session和cookie需要该项配置)
```xml
<mvc:interceptors>
        <!-- 国际化拦截器 -->
		<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
		<!-- 登录拦截器 -->
		<mvc:interceptor>
			<mvc:mapping path="/**" />
			<bean class="com.cisdi.ecis.pbs.interceptor.LoginInterceptor" />
		</mvc:interceptor>
</mvc:interceptors>
```
3. 提供一个controler方法,用于切换语言.
```java
    @RequestMapping("/lang")
    @ResponseBody
    public String lang(HttpServletRequest request) {
        String langType = request.getParameter("langType");
        if (langType.equals("zh")) {
            Locale locale = new Locale("zh", "CN");
            request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, locale);
        } else if (langType.equals("en")) {
            Locale locale = new Locale("en", "US");
            request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, locale);
        } else
            request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, LocaleContextHolder.getLocale());
        return null;
    }
```
4. 前端和之前一样,运行测试
<center>![中文环境](http://kiritor.github.io/img/i18n_01.png)</center>
可以看出当前浏览器语言顺序为中文优先,使用"切换语言"菜单手动切换系统语言为英文,之后刷新页面,界面还是为英文,当此session会话结束之后,刷新界面为中文.
<center>![英文环境](http://kiritor.github.io/img/i18n_02.png)</center>
到此为止,基于session的国际化配置完结,基于session的国际化是相对基于浏览器语言来说是比较常用的一种方式.
##基于Cookie的国际化实现:
大多数情况下国际化都是基于Cookie的,用户在设置完一次语言信息之后,之后自己没有修改,就算退出再登录,语言也是之前设定的.除非用户自己手动清空.这种就要通过Cookie来实现了.
1. 配置国际化资源文件(参考基于浏览器国际化配置)
2. 配置本地化信息解析策略,这里将基于Session的该项配置替换为如下即可:
```xml
<bean id="localeResolver"
		class="org.springframework.web.servlet.i18n.CookieLocaleResolver" />
```
3. SpringMVC配置解析拦截器,我们需要手动切换系统的语言(session和cookie需要该项配置),参考基于session的配置即可.
4. 提供一个controler方法,用于切换语言.
```java
    @RequestMapping("/lang")
    @ResponseBody
    public String lang(HttpServletRequest request,HttpServletResponse response) {
        String langType = request.getParameter("langType");
        if (langType.equals("zh")) {
            Locale locale = new Locale("zh", "CN");
            //request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, locale);
            new CookieLocaleResolver().setLocale(request, response, locale);
        } else if (langType.equals("en")) {
            Locale locale = new Locale("en", "US");
            //request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, locale);
            new CookieLocaleResolver().setLocale(request, response, locale);
        } else
        	new CookieLocaleResolver().setLocale(request, response, LocaleContextHolder.getLocale());
            //request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, LocaleContextHolder.getLocale());
        return null;
    }
```
5. 运行测试
<center>![中文环境](http://kiritor.github.io/img/i18n_01.png)</center>
可以看出当前浏览器语言顺序为中文优先,使用"切换语言"菜单手动切换系统语言为英文,之后刷新页面,界面还是为英文,当关闭此页面结束session会话,重新打开系统时界面仍为英文
<center>![英文环境](http://kiritor.github.io/img/i18n_02.png)</center>
而且debug可以看见cookie如图:
<center>![cooie](http://kiritor.github.io/img/cookie.jpg)</center>
这里需要多提及一点的是,关于基于cookie国际化的第二点,有3个属性可以配置:
```xml
<bean id="localeResolver"
		class="org.springframework.web.servlet.i18n.CookieLocaleResolver">
		<!-- 设置cookieName名称，可以根据名称通过js来修改设置，默认的名称为 类名+.LOCALE如上图 -->
		<property name="cookieName" value="lang" />
		<!-- 设置最大有效时间，如果是-1，则不存储，浏览器关闭后即失效，默认为Integer.MAX_INT -->
		<property name="cookieMaxAge" value="100000" />
		<!-- 设置cookie可见的地址，默认是“/”即对网站所有地址都是可见的，如果设为其它地址，则只有该地址或其后的地址才可见 -->
		<property name="cookiePath" value="/" />
</bean>
```
##基于ULR请求的国际化实现
这是一种比较有意思的情况,其实大多数时候不会有这种需求,类似一些常见的天气API,URL后面附带不同的语言的出的数据语种也不一样.不过天气API的实现方式可能有所不同.
1. 首先要扩展AcceptHeaderLocaleResolver类:
```java
package com.lcore.hr.utils;

import java.util.Locale;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

public class MyAcceptHeadersolver extends AcceptHeaderLocaleResolver{
	
	 public Locale resolveLocale(HttpServletRequest request) {
	        HttpSession session=request.getSession();
	        Locale locale=(Locale)session.getAttribute("locale");
	        if (locale==null){
	            session.setAttribute("locale",request.getLocale());
	            return request.getLocale();
	        }else{
	            return locale;
	        }

	    }

	    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
	        request.getSession().setAttribute("locale",locale);
	    }

}

```
2. 配置本地化信息解析器为我们自己自定义的解析器
```xml
<bean id="localeResolver"
		class="com.lcore.hr.utils.MyAcceptHeadersolver">
</bean>
```
3. 运行测试
输入地址:http://localhost:9999/hr/auth/user/listView?locale=zh_CH 界面显示为中文
<center>![中文环境](http://kiritor.github.io/img/i18n_01.jpg)</center>
输入地址:http://localhost:9999/hr/auth/user/listView?locale=en 界面显示为英文
<center>![英文](http://kiritor.github.io/img/i18n_02.jpg)</center>
OK,到此关于Spring MVC配合Velocity国际化配置也就到此为止了.
项目地址:[https://github.com/kiritor/hr](https://github.com/kiritor/hr)
需要说明的是,该项目使用的是基于Cookie的国际化配置,其他方式参考本文