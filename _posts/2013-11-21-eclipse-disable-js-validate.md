---
layout: post
title: eclipse彻底禁用js校验
description: eclipse彻底禁用js校验
---
在eclipse里禁止所有validate，导入maven项目后依旧会有JS校验

最终在.project会有
<pre class="prettyprint"><xmp>
<buildCommand>
	<name>org.eclipse.wst.jsdt.core.javascriptValidator</name>
	<arguments>
	</arguments>
</buildCommand>
</xmp></pre>

禁止用方法
import>Plugin-ins and Fragments
查找wst.jsdt.core,加入

org.eclipse.wst.jsdt.internal.core.JavaProject.java
查找addToBuildSpec 的调用,注释掉

最终用jar uvf org.eclipse.wst.jsdt.core_xx.jar org替换jar