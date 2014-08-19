---
layout: post
title: svnkit使用现有认证
description: svnkit使用现有认证
---
###svn
网上代码都是使用用户名密码的
官方让看svnkit-cli里的 SVNCommandEnvironment
最终代码
<pre class="prettyprint"><xmp>
SVNCommandEnvironment s = new SVNCommandEnvironment("", System.out, System.out, System.in);
SVNClientManager m = s.createClientManager();
SVNRepository r=m.createRepository(SVNURL.parseURIEncoded(""),false);
</xmp></pre>
 