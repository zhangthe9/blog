---
layout: post
title: 去除JSP页面自动生成的空行
categories:
- jsp
description:
- 去除JSP页面自动生成的空行
---
客户端view source看JSP生成代码时，空行由<%...%>后的回车换行而生成的
即每一行由<%...%>包含的JSP代码到客户端都变成一个空行，虽然不影响浏览，但还是希望能把他们删掉

###逐文件
JSP 2.1+ 
<pre class="prettyprint">
<xmp><%@page trimDirectiveWhitespaces="true" %>
</xmp></pre>

###servlet 2.5+, 即web.xml的XSD版本2.5
web.xml
<pre class="prettyprint">
<xmp><jsp-config>
	<jsp-property-group>
		<url-pattern>*.jsp</url-pattern>
		<trim-directive-whitespaces>true</trim-directive-whitespaces>
	</jsp-property-group>
</jsp-config>
</xmp></pre>
 

###init-param
web.xml
<pre class="prettyprint">
<xmp><init-param>
        <param-name>trimSpaces</param-name>
        <param-value>true</param-value>
</init-param></xmp></pre>