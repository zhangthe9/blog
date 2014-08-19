---
layout: post
title: Struts 1自身的一些缺点
description:
- Struts 1自身的一些缺点
categories:
- struts
- java
tags:
- struts
---
框架本身设计问题，如ActionForm,需要编写的代码过多，容易引起"类爆炸"
单元测试困难 
Struts 2.0的诞生解决了上述一些问题

###Struts2与Struts1的对比 
####Action 类  
- Struts1要求Action类继承一个抽象基类  
Struts1的一个普遍问题是使用抽象类编程而不是接口
如(Dispatch,Forward,Include,LookupDispatch,Mapping,Switch,Download)  Action
- Struts 2 Action类可以实现一个Action接口，也可实现其他接口  
使可选和定制的服务成为可能。Struts2提供一个ActionSupport基类去实现常用的接口
Action接口不是必须的，任何有execute标识的POJO对象都可以用作Struts2的Action对象
	
####线程模式
- Struts1 Action是单例模式并且必须是线程安全的，因为仅有Action的一个实例来处理所有的请求  
    单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心  
    Action资源必须是线程安全的或同步的。 
- Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题  
（servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题） 

####Servlet 依赖:
- Struts1 Action 依赖于Servlet API 
	Action调用时HttpServletRequest 和 HttpServletResponse 被传递给execute方法
- Struts 2 Action不依赖于容器，允许Action脱离容器单独被测试
	如果需要，Struts2 Action仍然可以访问初始的request和response  
	但是，其他的元素减少或者消除了直接访问HttpServetRequest 和 HttpServletResponse的必要性

####可测性: 
- 测试Struts1 Action的一个主要问题是execute方法暴露了servlet API（这使得测试要依赖于容器）  
	一个第三方扩展－－Struts TestCase－－提供了一套Struts1的模拟对象（来进行测试）
- Struts 2 Action可以通过初始化、设置属性、调用方法来测试，“依赖注入”支持也使测试更容易

####捕获输入
- Struts1 使用ActionForm对象捕获输入  
	所有ActionForm必须继承一个基类。其他JavaBean不能用作ActionForm，开发者经常创建多余的类捕获输入  
	动态Bean（DynaBeans）可以作为创建传统ActionForm的选择  
	但是，开发者可能是在重新描述(创建)已经存在的JavaBean（仍然会导致有冗余的javabean）
- Struts 2直接使用Action属性作为输入属性，消除了对第二个输入对象的需求  
	输入属性可能是有自己(子)属性的rich对象类型  
	Action属性能够通过web页面上的taglibs访问  
	Struts2也支持ActionForm模式。rich对象类型，包括业务对象，能够用作输入/输出对象  
	这种ModelDriven 特性简化了taglib对POJO输入对象的引用
	
####表达式语言
- Struts1 整合了JSTL，因此使用JSTL EL。这种EL有基本对象图遍历，但是对集合和索引属性的支持很弱
- Struts2可以使用JSTL，但是也支持一个更强大和灵活的表达式语言-(OGNL)

####绑定值到页面（view）
- Struts 1使用标准JSP机制把对象绑定到页面中来访问
- Struts 2 使用 "ValueStack"技术，使taglib能够访问值而不需要把你的页面（view）和对象绑定起来  
ValueStack策略允许通过一系列名称相同但类型不同的属性重用页面（view）。 
　 
####类型转换： 
• Struts 1 ActionForm 属性通常都是String类型。Struts1使用Commons-Beanutils进行类型转换  
每个类一个转换器，对每一个实例来说是不可配置的。 
• Struts2 使用OGNL进行类型转换。提供基本和常用对象的转换器

####校验： 
- Struts 1支持在ActionForm的validate方法中手动校验，或者通过Commons Validator的扩展来校验  
	同一个类可以有不同的校验内容，但不能校验子对象
- Struts2支持通过validate方法和XWork校验框架来进行校验  
	XWork校验框架使用为属性类类型定义的校验和内容校验，来支持chain校验子属性

####Action执行的控制： 
- Struts1支持每一个模块有单独的Request Processors（生命周期），但是模块中的所有Action必须共享相同的生命周期 
- Struts2支持通过拦截器堆栈（Interceptor Stacks）为每一个Action创建不同的生命周期  
	堆栈能够根据需要和不同的Action一起使用


struts2基本和webwork2一样，核心基本没改，修改了包名，说是struts和webwork合并，实则webwork利用struts招牌寻找市场

###struts2对webwork的改进之处
1.webwork.properties文件中变量现在可以配置在下列文件(struts2自带文档)
   struts-default.xml
   struts-plugin.xml(在每个plugin的jar包中)
   struts.xml(用constant元素)
   struts.properties(没有默认文件,可选,与webwork.properties文件相似,每个项目可以有这个文件)
   web.xml


2.加入了通配符的支持,这是在ww2中没有的,如下:
<pre class="prettyprint"><xmp><action name="*" >
       <result>/tutorial/{1}.jsp</result>
</action></xmp></pre>

Struts 2 是通过过滤器而非Servlet转发
web.xml中
<pre class="prettyprint"><xmp><?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_9" version="2.4"
	xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
	http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

	<filter>
		<filter-name>struts2</filter-name>
		<filter-class>
			org.apache.struts2.dispatcher.FilterDispatcher
		</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>
</web-app>
</xmp></pre>
解剖Actions
Struts1的Action
<pre class="prettyprint">	
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

public class MyAction extends Action {
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) {
		return mapping.findForward("success");
	}
}
</pre>

###Struts 2中的Action
不再继承于任何类或需要实现任何接口
任何声明为public String methodName() 方法都能通过配置来调用Action
返回的对象不是ActionForward，而是String。或Helper接口Action可以以常量方式提供常见结果
如“success”、“none”、“error”、“input”和“login”
Action过程中调用的方法（如execute()方法）是不带参数的。参数通过IOC取得

####Struts2配置文件 
web.xml，struts.xml，struts.properties，struts-default.xml，velocity.properties，struts-default.vm
web.xml，struts.xml必须，其它可选
	
web.xml：包含所有必须的框架组件的web部署描述符
Struts.xml:配置包含result/view类型、action映射、拦截器等的Struts2的主要配置文件
Struts.properties：配置struts2的框架属性
Struts-default.xml：在文件在struts-action-x.x.jar中，该文件是应该被包含在struts.xml中的缺省配置
velocity.properties：重写了velocity的配置文件
Struts-default.vm：相对于velocity的缺省配置

####Struts2配置元素
Struts2核心的配置文件是缺省的struts.xml
必要的时候，缺省的配置文件可以包含其它的配置文件；struts文件可以放入jar中，并自动插入应用程序
这样每个模块可以包含自己的配置文件并自动配置
在Freemarker和Velocity模块中，模板也能从classpath中加载，所以整个模块可以作为一个简单的jar文件被包含
Struts.xml配置文件可以包含Interceptor、Action类和Results

####Struts.xml配置元素说明
1、Packages 
Packages:packages把actions、results、results types、interceptors和interceptor-stacks组装到一个逻辑单元中
从概念上讲，packages就像一个对象，可以被其它子包重写，而且可以拥有自己独立的部分
Name属性是packages的必填元素，作为一个关键字被后边的包引用
extends元素可选，它允许包扩展一个和多个前边定义的包
注意
struts.xml文件是至上而下处理的，所有被扩展的包，需要在扩展包前定义
Abstract元素是可选的，它可以申明一个不包含actions的配置文件

2、Namespace
Namespace元素把actions细分到逻辑模块，每一个namespace都有自己的前缀（prefix）
namespace避免了action之间的名字冲突，当前缀出现在URI中时，这些标签都是名字空间感知的（"namespace aware"）
所以这些namespace prefix不必嵌入到表单或连接中
Default的namespace是一个空字符串“”，如果在指定的配置文件中，没有找到action，缺省的namespace也会被查找
Local/global策略允许应用程序在action “extends”元素层次结构之外，有全局的action配置
缺省的namespace也可以不在package中申明
Namespace prefix可以注册为java的申明式安全，以确保授权的用户才能访问namespace的资源
Root namespace("/")也被支持，root就是当直接请求context path的时候的namespace

namespace的用法就像struts1.x中的path一样，只不过它在package中什么路径，而在action中申明action名子罢了

3、Include 
Include元素使得框架能应用“divide and conquer”来配置文件。被include
的每个配置文件必须和struts.xml有一样的格式，一个大的项目可以采用这样方式来组织程序模块
Include元素也可以和package交替出现，框架将按照顺序加载配置文件

4、Interceptor configuration 
Interceptor允许应用程序在Action方法执行前后定义执行代码， 
Interceptor在应用程序开发中十分重要，对于Interceptor有许多用例，如validation, property population, security, logging, 和profiling
Interceptor被定义为一个Java类，Interceptor也可以组装成Interceptor-stack，将按照定义的顺序执行
在struts-default.xml中定义了一些缺省的Interceptor-stack,以便框架能很好地运行

5、Action 
Action是框架的“工作单元”。Action可以指定一个Interceptor-stack、 
一序列的return type和一序列的异常处理，但只有name属性是必须的。

Struts1 标签
<pre class="prettyprint">
<%@ taglib uri="http://struts.apache.org/tags-bean" prefix="bean" %>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://struts.apache.org/tags-logic" prefix="logic" %>
<%@ taglib uri="http://struts.apache.org/tags-tiles" prefix="tiles" %>
</pre>
Struts2 标签
<pre class="prettyprint">
<%@ taglib uri="/struts-tags" prefix="s" %>
</pre>



