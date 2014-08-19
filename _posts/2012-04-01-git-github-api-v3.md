---
layout: post
title: github API v3 使用
description:
- github API v3 使用
categories:
- Git
tags:
- github API
---
想用github api提供的功能建个 repo
http://developer.github.com/v3/repos/

命令格式: curl -d "post的数据" url

windows版curl引号处理要把 " 换成 \",而且整个命令要写成一行

<pre class="prettyprint">
curl -d "{\"name\": \"库名\" }"  -u "用户名:密码" https://api.github.com/user/repos -v
</pre>

##删除
<pre class="prettyprint">
curl -X DELETE -u "xx:xx" https://api.github.com/repos/用户/库 -v
</pre>


git clone https://github.com/用户名/库名.git