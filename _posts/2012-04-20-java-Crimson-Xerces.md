---
layout: post
title: Crimson和Xerces恩仇录 
description:
- Crimson和Xerces恩仇录 
categories:
- Java
tags:
- 
---
####Crimson
Sun捐给Apache的ProjectX项目
####Xerces
IBM捐给Apache的XML4J项目

Xerces胜，成Apache XML小组全力开发的XML API
Crimon早不做了，Xerces名满天下，到处是Xerces的DOM和SAX解析器

Sun不服气，JDK1.4用过时的Crimson
IBMJDK的XML解析器是Xerces

由于JDK的Class Loader优先级关系，即使Xerces包引入CLASSPATH，JDK还是用 Crimson，通JVM的verbose参数可以观察到

在JRE\lib\建jaxp.properties替换解析器
<pre class="prettyprint">
javax.xml.parsers.DocumentBuilderFactory=
org.apache.xerces.jaxp.DocumentBuilderFactoryImpl 
javax.xml.parsers.SAXParserFactory=
org.apache.xerces.jaxp.SAXParserFactoryImpl
</pre>

###JAXP的姗姗来迟
Sun在XML领域失利，之后XML的API已满天飞，Sun是规范的制订者，参考W3C标准制订JAXP规范

JAXP不像Xerces和Crimon，只是spec，本身不做任何事，提出统一接口，让其它XML API遵循JAXP编程，底层的API可以任意切换

JAXP包括几个工厂类，JDK1.4的javax.xml.parsers 包，寻找符合DOM标准的XML API实现类的位置
还包括一套interface，JDK1.4面的org.w3c.dom包。工厂类负责加载DOM的实现类

加载规则
工厂类首先根据命令行传入参数寻找，然后JRE\lib \jaxp.properties，都找不到就用Crimson


###不是标准的dom4j和jdom 
W3C的DOM标准API难用，于是便于使用的jdom出现，接着dom4j性能更好

jdom 相当于上面的 JAXP接口 ＋ Xerces DOM实现部分，本身没解析器，可用Xerces或Crimson解析器
jdom应用程序 -> jdom API -> Xerces/Crimson解析器

dom4j和jdom类似，但自己绑定了叫Alfred2的解析器，功能不是很全，但速度快，当没其它解析器时使用Alfred2解析器

dom4j应用程序 -> dom4j API ->  Xerces/Crimson解析器
或者 
dom4j应用程序 -> dom4j API ->  Alfred2解析器

dom4j.jar不含Alfred2解析器，dom4j-full.jar

###小插曲 
Sun是JAXP标准的制订者，甚至很执著的在JDK1.4里面绑定Crimson DOM实现和解析器，然后可笑的是，Sun自己的JAXM RI竟然不是用JAXP写出来的，而是dom4j，制订标准让大家遵守，自己却监守自盗

Hibernate也是dom4j读取XML配置，
JAXP RI 1.2开始，sun已经把Crimson换成了 Xerces 2
