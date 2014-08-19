---
layout: post
title: JSP/Servlet 几个编码的作用
description:
- JSP/Servlet 几个编码的作用
categories:
- java
tags:
- jsp
---
JSP/Servlet主要有以下几个地方可以设置编码：
pageEncoding="UTF-8"
contentType="text/html;charset=UTF-8"
request.setCharacterEncoding("UTF-8")
response.setCharacterEncoding("UTF-8")

前两个只能用于JSP，后两个可以用于JSP和Servlet

1、pageEncoding
设置JSP编译成Servlet时使用的编码
JSP先被编译成Servlet,pageEncoding告诉JSP编译器将JSP文件编译成Servlet时使用的编码
通常JSP内部定义的字符串（直接在JSP中定义，而不是从浏览器提交的数据）出现乱码时由此引起
例如JSP以GBK保存，而JSP中却指定pageEncoding="UTF-8"，就会引起JSP内部定义的字符串为乱码
	
另外，该参数还有一个功能，就是在JSP中不指定contentType参数，也不使用response.setCharacterEncoding时，指定对服务器响应进行重新编码的编码

2、contentType="text/html;charset=UTF-8"的作用是指定对服务器响应进行重新编码的编码。 
不使用response.setCharacterEncoding时，用该参数指定对服务器响应进行重新编码的编码。Tomcat默认是iso-8859-1

3、request.setCharacterEncoding("UTF-8")设置对客户端请求进行重新编码的编码 
该方法用来指定对浏览器发送来的数据进行重新编码（或者称为解码）时，使用的编码

4、response.setCharacterEncoding("UTF-8")指定对服务器响应进行重新编码的编码
服务器在将数据发送到浏览器前，对数据进行重新编码时，使用的就是该编码。 

###浏览器怎么样对接收和发送的数据进行编码
####response.setCharacterEncoding
response.setCharacterEncoding("UTF-8")指定对服务器响应进行重新编码的编码
同时浏览器根据这个参数来对其接收到的数据进行重新编码（或者称为解码）
所以在无论JSP设置response.setCharacterEncoding("UTF-8")或者("GBK")，浏览器均能正确显示中文（前提是发送到浏览器的数据编码是正确的，比如正确设置了pageEncoding参数等）

实验，JSP设置response.setCharacterEncoding("UTF-8")，IE-查看-编码是 Unicode（UTF-8）
response.setCharacterEncoding("GBK")是简体中文（GB2312）
	
###浏览器收发数据编码
对URL和参数会进行URL编码，对参数中的中文，浏览器使用response.setCharacterEncoding参数进行URL编码
以百度和GOOGLE为例，如搜索"汉字"
百度其编码"%BA%BA%D7%D6"
http://www.baidu.com/s?wd=%BA%BA%D7%D6
GOOGLE编码为"%E6%B1%89%E5%AD%97"
http://www.google.com.hk/search?q=%E6%B1%89%E5%AD%97

百度response.setCharacterEncoding参数为GBK，GOOGLE为UTF-8

浏览器接收server数据和发送数据到server时使用编码相同，默认均为JSP页面的response.setCharacterEncoding参数（或者contentType和pageEncoding参数），称其为浏览器编码
当然，可以修改浏览器编码(查看-编码)，但通常修改后会使原本正确的页面中出现乱码

一个有趣的例子是，在IE中浏览GOOGLE的主页时，将浏览器编码修改为"简体中文（GB2312）"，此时，页面上的中文会变成乱码，不理它，在文本框中输入"汉字"，提交，GOOGLE会将其编码为"%BA%BA%D7%D6"，可见，浏览器在对中文进行URL编码时，使用的就是浏览器编码

###服务器收发数据编码
发送数据，server按照response.setCharacterEncoding—contentType—pageEncoding的优先顺序，对要发送的数据进行编码

接收数据分三种情况
- 浏览器直接用URL提交的数据
- 表单的GET
- 表单的POST 方式提交的数据 

各种WEB服务器对这三种方式的处理也不相同，以Tomcat5.0为例 

无论使用那种方式提交，如果参数中包含中文，浏览器都会使用当前浏览器编码对其进行URL编码

对于表单中POST方式提交的数据，只要在接收数据的JSP中正确request.setCharacterEncoding参数，即将对客户端请求进行重新编码的编码设置成浏览器编码，就可以保证得到的参数编码正确

如何得到浏览器编码？上面提过，默认，浏览器编码就是在响应该请求的JSP页面中response.setCharacterEncoding设置的值
所以对于POST表单提交的数据，在获得数据的JSP页面中request.setCharacterEncoding要和生成提交该表单的JSP页面的response.setCharacterEncoding设置成相同的值 

对于URL提交的数据和表单中GET方式提交的数据，在接收数据的JSP中设置request.setCharacterEncoding参数是不行的，因为在Tomcat5.0中，默认情况下使用ISO-8859-1对URL提交的数据和表单中GET方式提交的数据进行重新编码（解码），而不使用该参数对URL提交的数据和表单中GET方式提交的数据进行重新编码（解码）

要解决该问题，应该在Tomcat的配置文件的Connector标签中设置useBodyEncodingForURI或者URIEncoding属性

useBodyEncodingForURI参数表示是否用request.setCharacterEncoding参数对URL提交的数据和表单中GET方式提交的数据进行重新编码，在默认情况下，该参数为false（Tomcat4.0中该参数默认为true）

URIEncoding参数指定对所有GET方式请求（包括URL提交的数据和表单中GET方式提交的数据）进行统一的重新编码（解码）的编码。URIEncoding和useBodyEncodingForURI区别是，URIEncoding是对所有GET方式的请求的数据进行统一的重新编码（解码），而useBodyEncodingForURI则是根据响应该请求的页面的request.setCharacterEncoding参数对数据进行的重新编码（解码），不同的页面可以有不同的重新编码（解码）的编码。所以对于URL提交的数据和表单中GET方式提交的数据，可以修改URIEncoding参数为浏览器编码或者修改useBodyEncodingForURI为true，并且在获得数据的JSP页面中request.setCharacterEncoding参数设置成浏览器编码

总结，Tomcat5.0，如何防止中文乱码
1、对于同一应用，最好统一编码，推荐为UTF-8，GBK也可以 
2、正确设置JSP的pageEncoding参数 
3、所有的JSP/Servlet中设置contentType="text/html;charset=UTF-8"或response.setCharacterEncoding("UTF-8")，从而间接实现对浏览器编码的设置
4、对于请求，可以使用过滤器或者在每个JSP/Servlet中设置request.setCharacterEncoding("UTF-8")。同时，要修改Tomcat的默认配置，推荐将useBodyEncodingForURI参数设置为true，也可以将URIEncoding参数设置为UTF-8（有可能影响其他应用，所以不推荐） 

###servlet 输出中文乱码
<pre class="prettyprint">
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { 
 PrintWriter pw = response.getWriter(); 
 response.setCharacterEncoding("utf-8"); 
 response.setContentType("text/html; charset=utf-8"); 
 pw.print("中文"); 
}
</pre>

无论编码改成gbk还是utf-8，页面访问到的一律是??

抓包，无论设为utf-8还是gbk抓到的均为
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=ISO-8859-1
Content-Length: 2
 
设置编码顺序调整，乱码问题解决

推断getWriter()返回的PrintWriter使用的charactor encoding是在这个函数返回时即已确定的，但到底是返回的PrintWriter内部属性还是运行时的控制，未找到依据

查看 tomcat中setCharacterEncoding方法的实现时发现如下代码：
<pre class="prettyprint">
public void setCharacterEncoding(String charset) { 

  if (isCommitted()) 
return; 

  // Ignore any call from an included servlet 
  if (included) 
return;

  // Ignore any call made after the getWriter has been invoked 
  // The default should be used 
  if (usingWriter) 
return; 

  coyoteResponse.setCharacterEncoding(charset); 
  isCharacterEncodingSet = true; 
 }
</pre>
其中usingWriter 标志为getPrinteWriter方法中设定，可见其控制逻辑为一旦返回了PrintWriter，本函数即不再生效。但是上述的推断没有进一步的证据

同时我们发现只有usingWriter标志，却没有usingOutputStream标记。猜测使用ServletOutputStream 输出不受此限制，经测试写出如下代码

ServletOutputStream out = response.getOutputStream(); 
out.print("中文"); 

//情况1：正常，浏览器按utf-8方式查看 
//response.setContentType("text/html; charset=utf-8"); 

//情况2：浏览器缺省按简体中文查看，手动设为utf-8方式查看正常 
//response.setCharacterEncoding("utf-8");

说明：这种方式不仅不需要在调用getOutputStream()之前设定字符集，甚至在print输出后设定都有效

查看setCharacterEncoding API文档，进一步发现：

Calling setContentType(java.lang.String) with the String of text/html and calling this method with the String of UTF-8 is equivalent with calling setContentType with the String of text/html; charset=UTF-8. 

原来只需要用response.setContentType("text/html; charset=utf-8"); 设定就ok，不需要两次调用。进一步

可反复设置，相互覆盖，据此写出如下测试代码


//情况1：正常，浏览器按utf-8方式查看 
response.setContentType("text/html; charset=gbk"); 
response.setCharacterEncoding("utf-8"); 
//情况2：正常，浏览器按简体中文方式查看 
//response.setContentType("text/html; charset=utf-8"); 
//response.setCharacterEncoding("gbk"); 

PrintWriter pw = response.getWriter(); 
pw.print("中文");

结论：

1.在servlet中输出中文，如果采用PrintWriter方式，需要在调用getPrintWriter()之前调用setContentType 或者 setCharacterEncoding；
采用ServletOutputStream方式，不受此限

2.setContentType 和 setCharacterEncoding两方法中设定characterEncoding的方法对服务器效果一致，不需要反复调用在输出文本内容时，采用response.setContentType("text/html; charset=utf-8");似乎更为方便