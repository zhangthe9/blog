---
layout: post
title: 语言的数据亲和力
description: 比较新的观点
categories:
- 语言
tags:
- 语言
---

Javascript、Perl、Python、Ruby、Groovy等越来越多地被熟悉和使用

C++、C#、Java等主流语言也在不断地融入函数式和动态性特征
      
程序员的百宝箱中可供选择的宝贝是越来多了，社区中关于语言间的比较和争论也更为热烈

常见“面向过程、面向对象”、"动、静态语言”、“命令、函数式范式”等比较

这类讨论关注点多集中于**设计相关话题**，如

动态语言的**Duck typing多态**和静态语言的**继承多态**的比较

**Prototype based**和**Class based**的比较 等

**数据处理**也是十分重要的方面，值得关注

本地信息存储,系统间信息交换都需要建立在一定的**数据格式基础**上。另外，不管语言属于那种范式，设计上采用什么模式，在微观层次上程序很大一部分工作都是在做数据处理

所以从数据处理角度比较和理解语言间差异有重要的现实意义。虽然数据通常是平台和语言无关的，但不同的语言在处理某种格式的数据时会表现出不同的难度，甚至某些数据格式只能采用特定的语言才能实现，这就是数据亲和力的不同

语言的数据亲和力(Data Affinity)指语言**与某种数据格式之间相容程度**，取决于语言的**数据模型，类型系统，以及库支持**等。语言对某种数据格式亲和力越强，操作某类数据越容易

----
- 二进制字节块格式

偏底层的操作系统、嵌入式和通信系统中，二进制字节块最常见

二进制数据布局紧凑、接近机器

但一般高级语言不方便直接和0101打交道，而是基于记录、结构体和类等结构化表示操作数据，存在底层二进制字节块和高层的结构化数据之间的转换问题

C语言作为最主要的系统语言具有很高的字节块数据亲和力

C，指针可以直接访问内存，还有结构体(struct)可以和字节块建立起直接的映射关系

例如Socket程序，通信双方事先定义共用的结构体

发送方先创建某结构体变量并填充字段，然后把变量对应的内存块copy到Socket

接收方从Socket读取字节块后把字节块强制类型转换为相应结构体指针即可读取个字段信息

整个过程中通信的双方都没有复杂的信息编码和解码的过程

<pre class="prettyprint">
struct t_data {

    int version;

    char type[10];

    float value;

};

//发送方

struct t_data data;

data.version = 1;

strcpy(data.type, “degree”);

data.value = 189.0;

send(socket,  &data, sizeof(data));

//接收方

struct t_data data;

read(socket,  &data, sizeof(data));

printf(“%d, %s, %f”, data.version, data.type, data.value);

</pre>
实际应用要注意内存对齐和大小端问题。内存对齐可用编译器预处理命令控制，保证内存中struct结构与传输的字节块具有相同的对齐方式；大小端问题需要通信的双方采用同样的大小端方式，否则就需要进行转换

C++可以完全兼容C的结构体，但C++的类(包括class和struct)中如果定义了虚函数，则会丧失结构的字节块数据亲和力，这是C++编程时需要权衡的一个因素

除了C/C++，其他语言中难以见到字节块数据亲和力，因为C/C++允许控制结构体/对象的内存布局，并允许对指针进行非类型安全的强制类型转换，Java，C#等不允许

所以，在Java、C#中进行字节块的编码解码就只能按照协议一个字段一个字段地按偏移量和长度进行解析。

C/C++的指针以及结构体和内存的直接映射带来了对字节块数据的亲和力，但同时也留下了内存访问和类型安全的隐患

而Java、C#在拥有引用安全和类型安全的同时也失去了对字节块数据的亲和力

 

- 文本格式

*nix 系统著名的管道

Bash文本处理例子：

1. 统计当前目录gz文件数目：

ls –l *.gz | wc –l

2. 分析统计log文件2011年6月26和27两天中每天中各页面的PV

cat service.log | grep ^2011-06-2[6-7] | cut –d ‘ ‘ –f 1, 3 | sort | uniq –c

	service.log:

	2011-06-25 13:00:55 /music/c.htm Safari

	…

	2011-06-26 08:01:23 /main.htm IE

	2011-06-26 08:03:01 /sports/b.htm Chrome

	…

	2011-06-27 11:41:06 /main.htm IE

	2011-06-27 11:52:41 /news/a.htm Firefox

输出:

	210 2011-06-26 /main.htm

	231 2011-06-26 /news/a.htm

	155 2011-06-26 /sports/b.htm

	288 2011-06-27 /main.htm

	292 2011-06-27 /news/a.htm

	161 2011-06-27 /sports/b.htm


上面的两个简单文本数据处理任务在C或C++实现麻烦得多，代码量多，加上编译调试，整个开发效率可能比Shell低一个数量级

除Shell外，Perl也是以强大的文本数据处理而闻名的
<pre class="prettyprint">
while (<STDIN>) {
    if (/hello\s(\w+)/i) {
        print “say hello to $1“
    }
    else if (/goodbye\s(\w+)/i) {
        print “say goodbye to $1”
    }
}
</pre>

输入：

HeLLo world

Goodbye bug

输出：

say hello to world

say goodbye to bug

Perl基于正则表达式的字符串处理不仅比C/C++等系统语言更强大，甚至比Python这样的动态语言也更强大和更方便，正则表达式是Perl语言的“一等公民”，使Perl比其他以库的方式支持正则表达式功能的语言具有更好的文本数据亲和力。Ruby也学习Perl把直接在语言上支持正则表达式

- 结构化文本格式

XML是通用（半）结构化文本数据交换格式。除具一般文本格式优点外，还有表达复杂的层次信息的优势

诞生以来就被大量用于配置文件和各种Web Service中

现代程序设计基本都少不了了XML打交道，但C++、Java和C#集中静态类型语言中处理XML并不轻松


Java解析和构建XML例子(不区 jdom,dom4j之类)：

<pre class="prettyprint">
<xmp><langs type="current">
	<language>Java</language>
	<language>Groovy</language>
	<language>JavaScript</language>
</langs>
</xmp></pre>

<pre class="prettyprint">
//解析

DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
try {
    DocumentBuilder db = dbf.newDocumentBuilder();
    Document doc = db.parse("src/languages.xml");
    Element langs = doc.getDocumentElement();
    System.out.println("type = " + langs.getAttribute("type"));
    NodeList list = langs.getElementsByTagName("language");
    for(int i = 0 ; i < list.getLength();i++) {
        Element language = (Element) list.item(i);
       System.out.println(language.getTextContent());
    }
}catch(Exception e) {
    e.printStackTrace();
}

//创建
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
try {
	DocumentBuilder db = dbf.newDocumentBuilder();
	Document doc = db.newDocument();
	Element langs = doc.createElement("langs");
	langs.setAttribute("type", "current");
	doc.appendChild(langs);
	Element language1 = doc.createElement("language");
	Text text1 = doc.createTextNode("Java");
	language1.appendChild(text1);
	langs.appendChild(language1);
	Element language2 = doc.createElement("language");
	Text text2 = doc.createTextNode("Groovy");
	language2.appendChild(text2);
	langs.appendChild(language2);
	Element language3 = doc.createElement("language");
	Text text3 = doc.createTextNode("JavaScript");
	language3.appendChild(text3);
	langs.appendChild(language3);
} catch (Exception e) {
    e.printStackTrace();
}
</pre>

冗长的Java代码

而实现同样的功能动态语言Groovy则十分简洁：
<pre class="prettyprint">
//Groovy解析XML               
def langs = new XmlParser().parse("languages.xml")
println "type = ${langs.attribute("type")}"
langs.language.each{
  println it.text()
}
//Groovy创建XML
def xml = new groovy.xml.MarkupBuilder()
xml.langs(type:"current"){
  language("Java")
  language("Groovy")
  language("JavaScript")
}
</pre>
Groovy操作XML的代码简洁而富有表达力，代码与XML几乎是一一对应的，如同直接在XML上进行操作的DSL一样，而相应的Java代码则看不到XML的影子

说明Groovy具有很高的XML数据的亲和力

两者差异呢原因在于Java要求所有的方法和属性都必须先定义再调用，严格的静态类型检查使得Java只能把XML元素作为“二等公民”来表达；而Groovy则没有静态类型检查的限制，可以自由地使用方法和属性来表达XML结构

Groovy创建XML例子中groovy.xml.MarkupBuilder类中实际上并没有langs, language这些方法，但会在调用的时候自动创建相应的XML结构

除XML外，JSON是另一种通用的半结构化的纯文本数据交换格式，常被视为轻量级的XML

JSON的本意是Javascript的对象表示(Javascript Object Notation)，属于Javascript语法子集，所以Javascript对JSON有原生的支持


Javascript中创建JSON对象例子：
<pre class="prettyprint">
	var json = {"langs" : {
		"type": "current",
		"language" : ["Java", "Groovy","Javascript"]
		}
	}
</pre>

许多Javascript程序通过AJAX从服务器获取JSON字符串，然后解析为JSON对象

Javascript解析JSON字符串可用通用的eval方式

`var json = eval("(" + jsonStr + “)");`

`alert(json.langs.type);`

甚至可以：

`eval("var json ="+ jsonStr);`

`alert(json.langs.type);`

不过eval的通用性带来一定的安全隐患，要注意数据源可信度，对不受信数据源可用专门的JSON解析库

Javascript具有很高的JSON数据亲和力

另外，Groovy 1.8也加入了对JSON的原生支持，操作JSON与Javascript一样方便

数据亲和力的话题还有很多值得探讨的，比如C#的Linq