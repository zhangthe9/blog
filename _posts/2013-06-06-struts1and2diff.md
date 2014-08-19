---
layout: post
title: struts1 2 区别
description: struts1 2 区别
categories:
- struts
tags:
- struts
---
###ActionForm
太多ActionForm容易引起“类爆炸”

###Action类
Struts1要求Action类继承一个抽象基类而不是接口

Struts2从概念上不在使用继承这种强耦合的方式实现对请求的处理，改为使用无耦合，或者接口耦合的方式。具体说来，移除ActionForm，和Action统一整合，Action的属性和以前ActionForm中的属性值有相同的效果，使Action成为一个POJO
任何有execute标识的POJO对象都可以用作Struts2的Action对象,也提供了ActionSupport基类去实现 常用的接口

Struts1 HttpServletRequest 和 HttpServletResponse 被传递给execute方法
Struts2 Action实现不再依赖与Servlet API，移除HttpRequest和HttpResponse等，简化单元测试

Action的execute方法不再返回ActionFoward这种框架偶度度很高的对象，直接采用字符串进行页面跳转的指向

###线程模型
Struts2的Action每一次请求创建实例，而不像Struts1中采用单例的方式进行处理
双刃剑，好处是降低了程序员处理并发程序的难度，因为实例本身的数据变成线程安全的了，坏处是增加了CPU压力，需要频繁进行对象的创建和GC

###表达式语言
struts1 用el 对集合和索引属性的支持很弱
struts2 OGNL

###绑定
struts1 用JSP
struts2 ValueStack技术，使标签库能够访问值，而不需要把对象和视图页面绑定在一起。支持OGNL进行类型转换，支持基本数据类型和常用对象之间的转换

Struts2移除WebWork自身的IoC框架，改使用Spring的

###web.xml
struts2 通过过滤器而非servlet转发
<pre class="prettyprint" >
<xmp><filter>
		<filter-name>struts2</filter-name>
		<filter-class>
			org.apache.struts2.dispatcher.FilterDispatcher
		</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping></xmp>
</pre>



###Struts1的Action
<pre class="prettyprint" >
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